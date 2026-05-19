# Resumen: Clase 7 - Memoria II - Locks

## Administración de Memoria: Kernel

### Mapeo directo en xv6:
- El kernel accede a RAM y registros de dispositivos usando direcciones virtuales = direcciones físicas.
- KERNBASE = 0x80000000 tanto en virtual como en físico.
- Excepciones: página trampolín (mapeada arriba del address space) y kernel stacks (mapeadas en dirección alta con página de protección debajo).

### kalloc(): Alocador de memoria física
- Estructura: lista libre de páginas físicas disponibles.
- Cada página libre almacena un `struct run { struct run *next; }` dentro de sí misma.
- Funciona como una pila (LIFO).

```c
void *kalloc(void) {
    acquire(&kmem.lock);
    r = kmem.freelist;
    if(r) kmem.freelist = r->next;
    release(&kmem.lock);
    if(r) memset((char*)r, 5, PGSIZE);
    return (void*)r;
}
```

- kalloc no recibe parámetros (siempre aloca 1 página).
- Opera con direcciones físicas, no virtuales.
- Se usa lock para proteger la región crítica de la lista.

### kfree():
```c
void kfree(void *pa) {
    r = (struct run*)pa;
    acquire(&kmem.lock);
    r->next = kmem.freelist;
    kmem.freelist = r;
    release(&kmem.lock);
}
```

### Mapeo de páginas (mappage):

- Objetivo: dada una dirección virtual, alocar una página física y mapearla.
- Se usan funciones auxiliares: `kalloc()`, `pte_init()`, `pte_pa()`.
- Proceso: obtener índices de la dirección virtual, verificar/crear tablas de páginas intermedias, crear la entrada final apuntando al frame físico.

### Páginas en xv6 (RISC-V):
- 3 niveles (x86 tiene 2).
- Los 25 bits superiores de la dirección virtual se ignoran.
- Bit V (Valid) indica si la página apunta a otra válida.

### Función walk en xv6:
```c
pte_t *walk(pagetable_t pagetable, uint64 va, int alloc) {
    for(int level = 2; level > 0; level--) {
        pte_t *pte = &pagetable[PX(level, va)];
        if(*pte & PTE_V)
            pagetable = (pagetable_t)PTE2PA(*pte);
        else {
            if(!alloc || (pagetable = kalloc()) == 0) return 0;
            memset(pagetable, 0, PGSIZE);
            *pte = PA2PTE(pagetable) | PTE_V;
        }
    }
    return &pagetable[PX(0, va)];
}
```

### brk():
- Ajusta el tamaño del heap sumando o restando bytes al program break.
- Uso típico: `mappage(p->pagetable, p->brk); p->brk += 4096;`

## Administración de Memoria: Usuario

### malloc():
- Devuelve bloque alineado a 8 bytes (double word).
- No inicializa la memoria.
- Usa `sbrk()` o `mmap()` internamente.

### Estructura del allocator (K&R):
- Header: `struct header { struct header *ptr; unsigned int size; };`
- Lista circular de bloques libres.
- Estrategia: buscar bloque libre lo suficientemente grande. Si no hay, pedir más memoria al OS (sbrk).
- Al liberar: insertar en lista libre y coalesce (fusionar bloques adyacentes).

### mmap():
```c
void *mmap(void *addr, size_t length, int prot, int flags, int fd, off_t offset);
int munmap(void *addr, size_t length);
```
- `MAP_ANONYMOUS`: mapping no respaldado por archivo, contenidos inicializados a cero.
- Se puede usar para pedir memoria al kernel.

## Concurrencia

### Race Conditions:
- Resultado depende del intercalado de operaciones de threads.
- Ejemplo clásico: dos threads haciendo `saldo += cantidad` sin sincronización → se pierden actualizaciones.
- Los compiladores y el procesador pueden reordenar instrucciones.

### Sección Crítica:
- Sección de código donde solo un thread puede ejecutar a la vez.
- Garantiza exclusión mutua y atomicidad.

## Locks

### Propiedades formales:
1. **Exclusión mutua**: a lo sumo un thread posee el lock.
2. **Progress**: si nadie tiene el lock y alguien lo quiere, alguno debe obtenerlo.
3. **Bounded waiting**: hay límite en la cantidad de veces que otros pueden obtener el lock antes que un thread que lo espera.

### API de Locks (pthreads):
```c
pthread_mutex_t lock;
pthread_mutex_init(&lock, NULL);
pthread_mutex_lock(&lock);
pthread_mutex_unlock(&lock);
pthread_mutex_trylock(&lock);
```

### Spinlock:
- Implementa busy wait: el thread gira en un loop verificando si el lock se liberó.
- Usa operación atómica `__sync_lock_test_and_set()`: lee valor actual, escribe nuevo valor, devuelve valor original. Todo atómicamente.
- `acquire()`: hace spin hasta obtener el lock.
- `release()`: usa `__sync_lock_release()` para liberar el lock.

```c
struct spinlock { uint locked; };

void acquire(struct spinlock *lk) {
    while(__sync_lock_test_and_set(&lk->locked, 1) != 0)
        ;
}

void release(struct spinlock *lk) {
    __sync_lock_release(&lk->locked);
}
```

#### Cuándo usar spinlock:
- Tiempo que se posee el lock es corto.
- Poca contention por los locks.
- Hay paralelismo real (múltiples CPUs).

### Sleeplock:
- El proceso se duerme si no logra conseguir el lock (cede el CPU).
- Al liberar el lock, se despiertan los procesos esperando.
- En xv6: se usa `sleep()` y `wakeup()`.

```c
struct sleeplock { uint locked; struct spinlock lk; };

void acquiresleep(struct sleeplock *lk) {
    acquire(&lk->lk);
    while(lk->locked) sleep(lk, &lk->lk);
    lk->locked = 1;
    release(&lk->lk);
}

void releasesleep(struct sleeplock *lk) {
    acquire(&lk->lk);
    lk->locked = 0;
    wakeup(lk);
    release(&lk->lk);
}
```

#### Cuándo usar sleeplock:
- La espera será larga (ej: esperar lectura de disco).
- Sistemas de un solo procesador (un spinlock nunca se liberaría).

### Lock Contention:
- Ocurre cuando múltiples threads intentan acceder a un recurso compartido simultáneamente y al menos uno está bloqueado esperando.
- La sección crítica debe ser lo más corta posible.

### Deadlock:
- Ocurre cuando dos o más threads se esperan mutuamente.
- Ejemplo: Thread 1 hace lock(a) → lock(b), Thread 2 hace lock(b) → lock(a). Ninguno avanza.
- Solución: establecer un orden fijo de adquisición de locks (ej: siempre lock(a) antes que lock(b)).

### Operaciones Atómicas:
- Instrucción que se ejecuta completamente o no se ejecuta ("todo o nada").
- No puede haber: lecturas parciales, interrupciones entre lectura y escritura, modificaciones por otros procesadores entre lectura y escritura.
- `__sync_lock_test_and_set(ptr, value)`: escribe value en *ptr, devuelve valor anterior. Atómica.
- `__sync_lock_release(ptr)`: libera el lock escribiendo 0 en *ptr.