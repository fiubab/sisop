# Resumen: Clase 4 - Scheduler I

## Transición User-Kernel en xv6

### Estructuras clave:
- **Trampoline**: código assembler para transicionar de user a kernel mode. Única página del kernel mapeada en el address space del usuario.
- **Trapframe**: donde el kernel guarda "la foto de los registros" antes de pasar a kernel mode. Se restaura al volver.

### Flujo de transición User → Kernel:
1. Se genera interrupción (ecall/INT 0x80).
2. El procesador salta al trampoline y cambia a Kernel Mode.
3. Se guardan todos los registros en el trapframe.
4. Se cambia satp (tabla de páginas) al del kernel.
5. Se apunta SP al kernel stack del proceso actual.
6. Se invoca `usertrap()`.

### Flujo de retorno Kernel → User:
1. Se invoca `userret()` en el trampoline.
2. Se cambia satp al del proceso.usuario.
3. Se restauran registros del trapframe.
4. Se invoca `sret` para volver al punto original.

## Context Switch

### En general:
- Se origina por: System Call, I/O Interrupt, Timer Interrupt, Exception.
- Si se decide hacer context switch → se invoca scheduler.
- El scheduler elige el próximo proceso → se ejecuta switch.

### En xv6 (2 context switches):
1. Proceso A → Scheduler (proceso de kernel sin user-space, uno por CPU).
2. Scheduler → Proceso B.

### Guardado de estado:

| Contexto | Donde se guarda |
|----------|----------------|
| User-space | `p->trapframe` (uno por proceso, mapeado en user address space) |
| Kernel en contexto del proceso | `p->context` (uno por proceso) |
| Kernel en contexto del scheduler | `c->context` (uno por CPU) |

## Scheduling

### Conceptos fundamentales:
- **Multiprogramación**: múltiples procesos listos, el SO intercala su ejecución.
- **Time Sharing**: compartir recurso computacional concurrentemente usando multiprogramación e interrupciones de reloj.
- **Time slice / Time quantum**: período de CPU otorgado a un proceso.
- **Workload**: carga de trabajo de los procesos.

### Clasificación de Schedulers:

| Tipo | Característica | Ejemplos |
|------|---------------|----------|
| **Interactivo** | Bajos tiempos de respuesta | Round Robin |
| **Batch** | Optimiza tiempo total | FIFO, SJF |

### Preemptivo vs No Preemptivo:
- **Preemptivo**: el SO puede interrumpir un proceso en ejecución.
- **No Preemptivo**: el proceso corre hasta terminar o ceder voluntariamente.

## Supuestos simplificadores (irreales):
1. Todos los procesos duran lo mismo.
2. Todos llegan al mismo tiempo.
3. Un job corre hasta completarse.
4. No hay I/O.
5. Se conoce el tiempo de ejecución de cada job.

## Métricas de Planificación

- **Turnaround Time** = T_completion - T_arrival (mide performance).
- **Response Time** = T_first_run - T_arrival (mide interactividad).

## Políticas de Scheduling

### FIFO (First In, First Out)
- El más simple. Se ejecutan en orden de llegada.
- Problema: **Convoy Effect**. Un job largo bloquea a los cortos.

### SJF (Shortest Job First)
- Ejecuta primero el job más corto.
- Mejora el turnaround time respecto a FIFO.
- Problema: asume que se conoce el tiempo de ejecución. Non-preemptive.

### STCF (Shortest Time-to-Completion First)
- Versión preemptive de SJF.
- Cuando llega un job nuevo, se desaloja el proceso actual y se elige el que le falta menos tiempo.
- Buen turnaround time, pero mal response time.

### Round Robin (RR)
- Cada proceso se ejecuta por un time slice, luego se pasa al siguiente.
- Time slice debe amortizar el context switch sin perder responsividad.
- Buen response time, mal turnaround time.

### Fórmula de utilización del CPU:
- Si un proceso pasa fracción `p` bloqueado en I/O, con `n` procesos: **Utilización = 1 - p^n**
- Ejemplo: p=0.8, n=3 → 1-0.8³ = 48.8%. n=10 → 89%.

## Round Robin en xv6

```c
c->proc = 0;
for(;;){
    for(p = proc; p < &proc[NPROC]; p++) {
        if(p->state == RUNNABLE) {
            p->state = RUNNING;
            c->proc = p;
            swtch(&c->context, &p->context);
            c->proc = 0;
        }
    }
}
```

## Considerando I/O

- Un proceso puede estar usando CPU o esperando I/O.
- Objetivo: siempre alguien usando el CPU.
- Procesos CPU-intensive compiten por CPU → timer los desaloja.
- Procesos I/O-intensive → susceptibles de delegar a coprocesadores (GPU) y volverse I/O intensive.

## Multi-Level Feedback Queue (MLFQ)

### Objetivos:
1. Optimizar turnaround time (ejecutar jobs cortos primero).
2. Minimizar response time para tareas interactivas.
3. Sin conocimiento a priori de la duración de los jobs.

### Reglas básicas:
- **Regla 1**: Si prioridad(A) > prioridad(B), A se ejecuta.
- **Regla 2**: Si prioridad(A) = prioridad(B), se ejecutan Round Robin.

### Primer Approach (Reglas 3, 4a, 4b):
- **Regla 3**: Al llegar, un job entra en la cola de mayor prioridad.
- **Regla 4a**: Si usa todo su time slice → baja un nivel de prioridad.
- **Regla 4b**: Si renuncia al CPU antes del time slice → se queda en el mismo nivel.

### Problemas del primer approach:
1. **Starvation**: muchos procesos interactivos pueden consumir todo el CPU, los largos nunca se ejecutan.
2. **Gaming**: un proceso puede hacer I/O antes del time slice para mantener prioridad alta.

### Segundo Approach (Regla 5 y regla 4 revisada):
- **Regla 5**: Después de un período S (boost), todas las tareas se mueven a la cola de mayor prioridad.
- Resuelve starvation: procesos largos eventualmente reciben CPU.
- Si un proceso largo se vuelve interactivo, el scheduler lo tratará como tal tras el boost.
- **Regla 4 revisada**: Una vez que un job consume su time slice total en un nivel (independientemente de cuántas veces cedió el CPU), su prioridad se reduce.

### Resumen final de reglas MLFQ:
1. Si prioridad(A) > prioridad(B), A se ejecuta.
2. Si prioridad(A) = prioridad(B), Round Robin.
3. Job nuevo → cola de mayor prioridad.
4. Un job que consume su time slice total → baja un nivel.
5. Periódicamente (cada S), boost de todas las tareas a la cola más alta.