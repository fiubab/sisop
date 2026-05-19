# Examen Práctico Nº1 - Sistemas Operativos (Fisop 2025)

---

## Sección 1: Preguntas de Opción Múltiple (2 pts c/u)

**1. ¿Cuál de las siguientes NO es una función del SO según la metáfora "Illusionist"?**
- a) Abstraer el hardware para simplificar el diseño de aplicaciones
- b) Ofrecer la ilusión de memoria casi infinita
- c) Gestionar y distribuir recursos entre aplicaciones
- d) Ofrecer la ilusión de uso exclusivo de procesadores

**2. En x86, la instrucción `INT 0x80` se usa para:**
- a) Generar una interrupción de hardware del timer
- b) Invocar una system call desde user mode
- c) Cambiar de kernel mode a user mode
- d) Cargar la Global Descriptor Table

**3. ¿Qué system call NO crea un nuevo proceso?**
- a) fork()
- b) execve()
- c) vfork()
- d) clone() con flags específicos

**4. En xv6, el trampoline se encarga de:**
- a) Guardar los registros del proceso cuando se hace context switch entre procesos
- b) Realizar la transición de user mode a kernel mode y viceversa
- c) Planificar el próximo proceso a ejecutar
- d) Gestionar la tabla de páginas del proceso

**5. En la transición user → kernel en xv6, ¿en qué estructura se guardan los registros del procesador?**
- a) `p->context`
- b) `c->context`
- c) `p->trapframe`
- d) En el stack del scheduler

**6. ¿Cuál es la métrica que mide cuánto tiempo tarda un trabajo en completarse desde que llega al sistema?**
- a) Response time
- b) Turnaround time
- c) Waiting time
- d) throughput

**7. En MLFQ, ¿qué sucede cuando un proceso utiliza todo su time slice?**
- a) Se mantiene en la misma cola de prioridad
- b) Se elimina del sistema
- c) Se reduce su prioridad (baja un nivel en la cola)
- d) Se aumenta su prioridad (sube un nivel)

**8. En CFS (Linux), ¿qué proceso se selecciona para ejecutar?**
- a) El de mayor nice value
- b) El de menor vruntime
- c) El más reciente en llegar
- d) El de mayor weight

**9. ¿Cuál estructura se usa en CFS para organizar los procesos?**
- a) Array ordenado
- b) Linked list
- c) Red-Black Tree
- d) Hash table

**10. En memoria paginada de x86 (32 bits, páginas de 4KB), ¿cuántos bits se usan para el offset dentro de una página?**
- a) 10 bits
- b) 12 bits
- c) 20 bits
- d) 22 bits

---

## Sección 2: Preguntas de Desarrollo Corto (3 pts c/u)

**1. Explique brevemente las diferencias entre spinlock y sleeplock. ¿En qué situaciones se debe usar cada uno?**

**2. ¿Qué es el Copy on Write (COW)? Explique su funcionamiento y qué problema resuelve.**

**3. Explique qué es un file descriptor y qué recursos puede representar en Unix. De un ejemplo.**

**4. ¿Cuál es la diferencia entre un hard link y un soft link? ¿Qué sucede con cada uno si se borra el archivo original?**

**5. ¿Qué es la TLB y por qué es necesaria? ¿Qué sucede con la TLB al hacer un context switch?**

---

## Sección 3: Ejercicio de Scheduling (5 pts)

Se tienen los siguientes procesos que llegan todos en t=0:

| Proceso | Tiempo de CPU necesario |
|---------|------------------------|
| A | 8 |
| B | 4 |
| C | 2 |
| D | 1 |

**a)** Calcular el turnaround time promedio con FIFO.
**b)** Calcular el turnaround time promedio con SJF.
**c)** Si se usa Round Robin con time slice = 1, calcular el response time promedio.

---

## Sección 4: Ejercicio de Memoria (5 pts)

Considere un sistema x86 con memoria virtual paginada de dos niveles, espacio de direcciones de 32 bits y páginas de 4KB.

Se tiene un array de 80,000 enteros (4 bytes c/u) que comienza en la dirección virtual `0x00A00000`.

**a)** ¿Cuántos bytes ocupa el array en total?
**b)** ¿Cuántas páginas de datos se necesitan?
**c)** ¿A cuántos frames de memoria física distintos necesita acceder el SO (contando tablas de páginas intermedias)?

---

## Sección 5: Ejercicio de File System (5 pts)

Se prepara el sistema con:
```
mkdir /dir
mkdir /dir/s
echo 'hola' > /dir/s/x
echo 'mundo' > /dir/s/y
```

**a)** ¿A cuántos inodos y bloques accede `cat /dir/s/x`?
**b)** Si se ejecuta `ln /dir/s/x /dir/s/z` y luego `rm /dir/s/x`, ¿a cuántos inodos y bloques accede `cat /dir/s/z`?
**c)** Si se ejecuta `ln -s /dir/s/y /dir/s/w`, ¿a cuántos inodos y bloques accede `cat /dir/s/w`?

---

## Sección 6: Ejercicio de Concurrencia (5 pts)

Se tiene el siguiente código que se ejecuta concurrentemente por dos threads:

```c
int contador = 0;

void incrementar() {
    for (int i = 0; i < 1000; i++) {
        contador++;
    }
}
```

**a)** ¿Cuál es el valor máximo posible de `contador` al finalizar?
**b)** ¿Cuál es el valor mínimo posible de `contador` al finalizar?
**c)** Modifique el código usando un mutex para garantizar que el resultado sea siempre 2000.

---

## Respuestas

### Sección 1:
1-c, 2-b, 3-b, 4-b, 5-c, 6-b, 7-c, 8-b, 9-c, 10-b

### Sección 3:
**a)** FIFO: A(8), B(8+4=12), C(12+2=14), D(14+1=15). Turnaround = (8+12+14+15)/4 = **12.25**
**b)** SJF: D(1), C(1+2=3), B(3+4=7), A(7+8=15). Turnaround = (1+3+7+15)/4 = **6.5**
**c)** RR con ts=1: Response times: A=0, B=1, C=2, D=3. Promedio = (0+1+2+3)/4 = **1.5**

### Sección 4:
**a)** 80000 × 4 = 320,000 bytes
**b)** 320000/4096 = 78.125 → **79 páginas**
**c)** Dirección 0x00A00000 = 0000000000 1010000000 000000000000 → PDE[0], PTE[640]
    Dirección final = 0x00A00000 + 320000 - 1 = 0x00A4E200
    = 0000000000 1010010001 000000000000 → PDE[0], PTE[657]
    1 PDE + (657-640+1) = 18 PTEs + 79 data pages = **98 frames**

### Sección 5:
**a)** /: inodo(1) + bloque(1) → dir: inodo(2) + bloque(2) → s: inodo(3) + bloque(3) → x: inodo(4) + bloque(4) = **4 inodos, 4 bloques**
**b)** Hard link: misma cantidad que (a) ya que apunta al mismo inodo. **3 inodos, 3 bloques** (/ → dir → z)
**c)** Symlink: /: inodo + bloque → dir: inodo + bloque → w: inodo(SYMLINK) + bloque(ruta) → /: inodo + bloque → dir: inodo + bloque → s: inodo + bloque → y: inodo + bloque = **7 inodos, 7 bloques**

### Sección 6:
**a)** 2000 (si no hay interleaving)
**b)** 2 (peor caso: ambos leen el mismo valor antes de escribir)
**c)** Agregar `pthread_mutex_lock(&mut)` antes de `contador++` y `pthread_mutex_unlock(&mut)` después.