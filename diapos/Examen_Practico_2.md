# Examen Práctico Nº2 - Sistemas Operativos (Fisop 2025)

---

## Sección 1: Preguntas de Opción Múltiple (2 pts c/u)

**1. ¿Cuáles son las 3 metáforas que describen las funciones de un SO?**
- a) Kernel, User Space, Hardware
- b) Referee, Illusionist, Glue
- c) CPU, Memoria, I/O
- d) Scheduling, Memory, File System

**2. ¿Qué modo de ejecución de CPU permite ejecutar instrucciones privilegiadas?**
- a) User mode (Ring 3 en x86)
- b) Supervisor mode (Ring 0 en x86)
- c) Ambos modos por igual
- d) Ninguno, las instrucciones privilegiadas se ejecutan en hardware

**3. ¿Cuál de las siguientes afirmaciones sobre `fork()` es FALSA?**
- a) Crea una copia exacta del proceso padre
- b) Devuelve 0 al proceso hijo
- c) Crea un nuevo proceso con un PID diferente
- d) El hijo no hereda los file descriptors del padre

**4. En xv6, un context switch consta de:**
- a) 1 cambio: proceso A → proceso B directamente
- b) 2 cambios: proceso A → scheduler → proceso B
- c) 3 cambios: proceso A → kernel → scheduler → proceso B
- d) Ninguno de los anteriores

**5. ¿Cuál es el propósito del bit P (Present) en una Page Table Entry?**
- a) Indicar si la página ha sido accedida
- b) Indicar si la página está presente en memoria física
- c) Indicar si la página es de solo lectura
- d) Indicar si la página es global

**6. En STCF (Shortest Time-to-Completion First):**
- a) Se ejecutan los procesos en orden de llegada
- b) Se puede desalojar (preempt) al proceso actual si llega uno con menor tiempo restante
- c) Los procesos interactivos tienen prioridad absoluta
- d) Se asigna un time slice fijo a cada proceso

**7. ¿Qué valor de nice otorga la mayor prioridad a un proceso en Linux?**
- a) 19
- b) 0
- c) -20
- d) 10

**8. En MLFQ, ¿cuándo se ejecuta el boost de prioridad?**
- a) Cuando un proceso termina su time slice
- b) Periódicamente (cada un tiempo S)
- c) Cuando un proceso hace I/O
- d) Cuando el scheduler lo decide individualmente

**9. ¿Qué tipo de dispositivo transfiere datos byte a byte?**
- a) Dispositivo de bloque
- b) Dispositivo de carácter
- c) Ambos
- d) Ninguno

**10. La syscall `brk()` se usa para:**
- a) Crear un nuevo proceso
- b) Ajustar el tamaño del heap del proceso
- c) Cerrar un file descriptor
- d) Leer de un archivo

---

## Sección 2: Preguntas de Desarrollo Corto (3 pts c/u)

**1. Explique la diferencia entre `execve()` y `fork()`. ¿Por qué se usan juntas en el patrón fork-exec?**

**2. Describa los 3 estados principales de un proceso y las transiciones posibles entre ellos. Dibuje el diagrama de estados.**

**3. ¿Qué es la segmentación de memoria? ¿Qué ventajas y desventajas tiene respecto a la paginación?**

**4. Explique qué es el VFS (Virtual File System) y qué problema resuelve.**

**5. ¿Qué es un deadlock? De un ejemplo con dos threads y dos locks. ¿Cómo se puede prevenir?**

---

## Sección 3: Ejercicio de Scheduling con I/O (5 pts)

Se tienen los siguientes procesos:

| Proceso | Llegada | Tiempo CPU | I/O |
|---------|---------|-----------|-----|
| A | 0 | 5 | Usa I/O al finalizar (1ms) |
| B | 0 | 3 | - |
| C | 2 | 2 | - |

**a)** Con FIFO, calcular el turnaround time promedio.
**b)** Con SJF non-preemptive, calcular el turnaround time promedio.
**c)** Si un proceso pasa el 80% del tiempo bloqueado en I/O, ¿cuál es la utilización del CPU con 5 de estos procesos?

---

## Sección 4: Ejercicio de Memoria Paginada (5 pts)

Considere un sistema x86 de 32 bits con paginación de dos niveles y páginas de 4KB.

Se tiene un programa cuyo segmento de código comienza en `0x00400000` y ocupa 18KB, y su segmento de datos comienza en `0x00500000` y ocupa 6KB.

**a)** ¿Cuántas páginas se necesitan para el segmento de código?
**b)** ¿Cuántas páginas se necesitan para el segmento de datos?
**c)** ¿Cuántos page directory entries (PDE) se necesitan en total?
**d)** ¿Cuántos page table entries (PTE) se necesitan en total?
**e)** Si cada PTE y PDE ocupa 4 bytes, ¿cuánta memoria ocupa la estructura de paginación completa?

---

## Sección 5: Ejercicio de File System (5 pts)

Diseñe un vsfs (Very Simple File System) para un disco de 2048 bloques de 4KB cada uno, donde cada inodo ocupa 256 bytes.

**a)** ¿Cuántos inodos caben en un bloque?
**b)** Usando el criterio de la cátedra (1 inodo por bloque del disco), ¿cuántos inodos se necesitan?
**c)** ¿Cuántos bloques se necesitan para la tabla de inodos?
**d)** ¿Cuántos bloques hay en la data region?
**e)** Dibuje el layout del filesystem indicando el número de bloque inicial de cada región.

---

## Sección 6: Ejercicio de Pipes y FDs (5 pts)

Considere el siguiente código:

```c
int fd[2];
pipe(fd);

if (fork() == 0) {
    close(fd[0]);
    dup2(fd[1], STDOUT_FILENO);
    close(fd[1]);
    execlp("ls", "ls", NULL);
} else {
    close(fd[1]);
    dup2(fd[0], STDIN_FILENO);
    close(fd[0]);
    execlp("wc", "wc", "-l", NULL);
}
```

**a)** ¿Qué hace este código? Describa el flujo de datos.
**b)** ¿Por qué es necesario cerrar `fd[0]` en el hijo y `fd[1]` en el padre?
**c)** ¿Qué pasaría si NO se cerraran los extremos no usados del pipe?

---

## Sección 7: Ejercicio de Traducción de Direcciones (5 pts)

En un sistema x86 con paginación de 2 niveles, páginas de 4KB y las siguientes tablas:

**Page Directory** (comienza en dirección física 0x1000):

| Índice | PDE |
|--------|-----|
| 0 | 0x2003 (Present, R/W) |
| 1 | 0x3003 (Present, R/W) |
| ... | ... |

**Page Table en 0x2000** (apuntada por PDE[0]):

| Índice | PTE |
|--------|-----|
| 0 | 0x4003 (Present, R/W) |
| 1 | 0x5003 (Present, R/W) |
| 2 | 0x6003 (Present, R/W) |
| ... | ... |

Traduzca la dirección virtual `0x00000840` a dirección física. Muestre todos los pasos.

---

## Respuestas

### Sección 1:
1-b, 2-b, 3-d, 4-b, 5-b, 6-b, 7-c, 8-b, 9-b, 10-b

### Sección 3:
**a)** A(0-5), B(5-8), C(8-10). Turnaround: A=5, B=8, C=8. Promedio = 21/3 = **7**
**b)** Orden: B(0-3), C(3-5), A(5-10). Turnaround: B=3, C=3, A=10. Promedio = 16/3 ≈ **5.33**
**c)** Utilización = 1 - 0.8⁵ = 1 - 0.32768 = **0.672 = 67.2%**

### Sección 4:
**a)** 18KB/4KB = 4.5 → **5 páginas**
**b)** 6KB/4KB = 1.5 → **2 páginas**
**c)** `0x00400000` → PDE[1], `0x00500000` → PDE[1] → **1 PDE**
**d)** Código: PTE[0..4] = 5 PTEs, Datos: PTE[0..1] = 2 PTEs → **7 PTEs**
**e)** 1 PDE × 4B + 1 PT (1024 × 4B) + ... = Page Directory = 1024×4 = 4096B, una Page Table = 4096B. Total ≈ **8192 B** (1 PD + 1 PT)

### Sección 5:
**a)** 4096/256 = 16 inodos por bloque
**b)** 2048 inodos (1 por bloque)
**c)** 2048/16 = 128 bloques de inodos
**d)** 2048 - 128 - 1 - 1 - 1 = **1917 bloques de datos**
**e)** Layout: [S:0][i:1][d:2][I:3-130][D:131-2047]

### Sección 6:
**a)** El hijo ejecuta `ls` y su salida se redirige al pipe. El padre lee del pipe y ejecuta `wc -l`. Equivale a `ls | wc -l`.
**b)** Para que el pipe funcione correctamente: el hijo escribe y el padre lee. Cerrar extremos no usados evita bloqueos.
**c)** Si no se cierran, `wc` nunca terminaría porque el pipe permanecería abierto (el hijo tendría fd[0] abierto, el padre fd[1]).

### Sección 7:
Virtual `0x00000840` = 0000000000 0000000010 000001000000
- PDE index: 0 → PDE = 0x2003 → Frame = 0x2000
- PTE index: 2 → PTE = 0x6003 → Frame = 0x6000
- Offset: 0x840 → 0x840 & 0xFFF = 0x840
- **Dirección física = 0x6000 + 0x840 = 0x6840**

Wait, let me recalculate:
Virtual `0x00000840` = 0000000000 0000000010 000001000000
- Bits 31-22 (PDE index): 0000000000 = 0
- Bits 21-12 (PTE index): 0000000010 = 2
- Bits 11-0 (offset): 000001000000 = 0x840? No...

`0x00000840 = 0000 0000 00 | 00 0000 1000 | 0100 0000 0000`
- PDE index: 0000000000 = 0
- PTE index: 0000000010 = 2
- Offset: 0100 0000 0000 = 0x440

PDE[0] = 0x2003 → Page Table en 0x2000
PTE[2] = 0x6003 → Frame en 0x6000
Dirección física = 0x6000 + 0x440 = **0x6440**