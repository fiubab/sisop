# Resumen: Clase 5 - Scheduler II

## Linux: Evolución del Scheduler

| Versión | Scheduler | Complejidad |
|---------|-----------|-------------|
| Hasta 2.4 | Round-Robin básico | O(n) |
| 2.6 | O(1) Scheduler | O(1) |
| 2.6.23 | CFS (Completely Fair Scheduler) | O(log n) |
| 6.6 | EEVDF | O(log n) |

## Completely Fair Scheduler (CFS)

### Principio básico:
- Si hay n procesos, cada uno debería recibir 1/n del tiempo de CPU.
- No asigna time slices fijos, sino tiempo proporcional al peso de prioridad.
- **Algoritmo de selección**: siempre elige el proceso con menor **vruntime**.

### vruntime (Virtual Runtime):
- Cada proceso tiene un contador vruntime = tiempo de ejecución ajustado por prioridad.
- `vruntime += delta_exec * (NICE_0_LOAD / weight)`
- Procesos con mayor prioridad (menos nice) → vruntime crece más lento → se seleccionan más frecuentemente.

### nice y pesos:
- Rango de nice: -20 (más prioritario) a 19 (menos prioritario). Default: 0.
- Cada valor de nice se mapea a un peso (weight).
- nice -10 → weight 1024, nice 0 → weight 256, nice 10 → weight 128 (valores de ejemplo).

### Timeslice dinámico:
- `timeslice = sched_latency * (weight_proceso / sum_weights)`
- `sched_latency`: período donde todos los procesos ejecutables corren al menos una vez (~20-48ms).
- `min_granularity`: tiempo mínimo de ejecución antes de ser desalojado (~10-20% de sched_latency).
- Si hay muchos procesos y sched_latency/n < min_granularity → se ajusta sched_latency.

### Runqueue:
- Cada CPU tiene una runqueue = Red-Black Tree ordenado por vruntime.
- El proceso con menor vruntime está más a la izquierda → selección O(log n).
- Inserción O(log n), eliminación O(log n), acceso al mínimo O(1).

### Funcionamiento detallado:
1. El scheduler selecciona el proceso con menor vruntime.
2. Se ejecuta hasta que su vruntime ya no es el mínimo, o se bloquea/termina quantum.
3. Proceso bloqueado → se elimina del árbol; al despertar → se reinserta con vruntime ajustado.
4. CFS también hace balance de carga entre CPUs.

### Group Scheduling:
- Primero se distribuye tiempo entre grupos (ej: usuarios), luego dentro de cada grupo.
- Un usuario con más procesos no obtiene más tiempo total de CPU.

## Threads

### Definición:
- Un thread es una secuencia de ejecución atómica que representa una tarea planificable.
- Características por thread: Thread ID, registros propios, stack propio, prioridad propia, errno propio, datos específicos.

### Thread vs Proceso:

| Thread | Proceso |
|--------|---------|
| Comparten memoria por defecto | No comparten memoria por defecto |
| Comparten FDs por defecto | No comparten FDs por defecto |
| Comparten contexto filesystem | No comparten contexto filesystem |
| Comparten manejo de señales | No comparten manejo de señales |

### Modelos de Threading:
- **Cooperativo**: no hay interrupción a menos que se solicite. Problemas: un thread puede monopolizar.
- **Preemptivo**: un thread en running puede ser movido en cualquier momento. Es el más usado.

### Thread Scheduler:
- Crea la ilusión de muchos threads con procesadores fijos.
- Desde el punto de vista del thread, cada instrucción se ejecuta inmediatamente.
- Distintos interleaveings posibles → no determinismo.

### API de Threads (pthreads):

```c
pthread_create(&thread, attr, start_routine, arg);  // Crear thread
pthread_join(thread, value_ptr);                       // Esperar thread
```

### Estructuras del SO para threads:

- **Thread Control Block (TCB)**: por cada thread. Contiene: puntero al stack, copia de registros, Thread ID, prioridad, estado.
- **Estado compartido**: código, variables globales, heap.

### Estados de un Thread:
| Estado | Descripción |
|--------|-------------|
| **Init** | Inicializando estructuras per-thread. |
| **Ready** | Listo para ejecutar, en ready list. |
| **Running** | Ejecutándose en el procesador. |
| **Waiting** | Esperando un evento, en waiting list. |
| **Finished** | Terminado, nunca más se ejecuta. |

### Threads en Linux: Modelo 1-1
- En el kernel no hay distinción entre thread y proceso → todo es una tarea ejecutable.
- Se usa `clone()` con distintos flags:
  - Thread: `clone(CLONE_VM | CLONE_FS | CLONE_FILES | CLONE_SIGHAND, 0)`
  - Proceso: `clone(SIGCHLD, 0)` (equivale a fork)
  - vfork: `clone(CLONE_VFORK | CLONE_VM | SIGCHLD, 0)`

### Copy on Write (COW):
- Al hacer fork(), el kernel no copia páginas de memoria.
- Ambos procesos comparten las mismas páginas físicas marcadas como solo lectura.
- Cuando uno intenta escribir → se genera page fault → se copia la página → se ajustan permisos.
- Optimización巨大: evita copiar memoria si el hijo hace exec() inmediatamente.