# Resumen: Clase 3 - El Proceso

## De Programa a Proceso

### Proceso de Compilación (4 fases):

| Fase | Herramienta | Descripción |
|------|-------------|-------------|
| **1. Preprocesador** | `cpp` | Reemplaza directivas (`#include`, `#define`). Produce texto puro. |
| **2. Compilador** | `cc1` | Análisis léxico, sintáctico, semántico, optimización, generación de código intermedio (IR). Produce `.s` (ensamblador). |
| **3. Ensamblador** | `as` | Convierte `.s` a código máquina. Produce `.o` (object file, formato ELF). Deja relocations pendientes. |
| **4. Enlazador** | `ld` | Resuelve símbolos, reubica direcciones, genera ELF final ejecutable. |

## Formato ELF

- Un archivo ejecutable contiene: instrucciones máquina, dirección de punto de entrada, datos, símbolos y tablas de reubicación, bibliotecas compartidas, otra información.
- Tiene dos lecturas paralelas: Program Header Table (para el loader, segmentos en tiempo de ejecución) y Section Header Table (para linker/compiler, secciones lógicas).

### Comandos útiles:
```
readelf -h <archivo>   # ELF Header
readelf -l <archivo>   # Program Header Table
readelf -S <archivo>   # Section Header Table
readelf -s <archivo>   # Tabla de símbolos
readelf -r <archivo>   # Relocation Table
objdump -d <archivo>   # Disassembly
xxd <archivo> | head   # Magic number
```

## Linker Script (.ld)

- Archivo de texto que indica al linker cómo construir el binario final: qué secciones, en qué orden, y en qué dirección de memoria virtual.
- El mapper/loader usa el ELF como "plano" para construir el proceso durante `execve()`.

## OpenSBI (RISC-V)

- OpenSBI = implementación de referencia de SBI (Supervisor Binary Interface).
- API que el firmware le ofrece al kernel.
- RISC-V define 3 niveles de privilegio: Machine, Supervisor, User.

## El Proceso: Definición

- "Abstracción que provee el Kernel para ejecución protegida" [DAH]
- "Programa en ejecución" [ARP]
- "Instancia de un programa en ejecución" [VAH]
- Incluye: archivos abiertos, señales pendientes, datos internos del kernel, estado del procesador, espacio de direcciones, hilos de ejecución.

## Estructura del Proceso en Memoria

| Segmento | Contenido |
|----------|-----------|
| **Text** | Instrucciones del programa (código). |
| **Data** | Variables globales (extern, static en C). |
| **Heap** | Memoria dinámica (crece hacia arriba). |
| **Stack** | Variables locales, trace de llamadas (crece hacia abajo). |

## Virtualización de Memoria

- Cada proceso cree tener toda la memoria para sí (direcciones virtuales).
- Todos los procesos comienzan en la dirección virtual 0.
- La MMU traduce direcciones virtuales a físicas.
- El Loader usa el ELF para armar el address space.

## System Call brk()

- Incrementa el break del programa → expande el heap.
- `brk()` es una syscall, opera con bloques grandes (páginas).
- `malloc()` se implementa en user space usando `brk()`, maneja estructuras propias para optimizar.

## Virtualización de Procesador

- El SO da la ilusión de un procesador exclusivo para cada programa.
- Se logra mediante context switch rápido (cientos de procesos en un solo procesador).

## Estados del Proceso

| Estado | Descripción |
|--------|-------------|
| **Running** | Ejecutándose en un procesador. |
| **Ready** | Listo para correr pero el SO no lo ejecuta aún. |
| **Blocked** | Esperando un evento (I/O, etc.). |

### Estados adicionales (System V):
- Running User Mode, Running Kernel Mode, Ready to Run in Memory, Asleep in Memory, Ready to Run but Swapped, Asleep Swapped, Preempted, Created, Zombie.

## Proceso: El Contexto

El contexto de un proceso es la unión de:

| Nivel | Contenido |
|-------|-----------|
| **User-level** | Text, Data, Stack, Heap (espacio de direcciones virtual). |
| **Register** | Program Counter, Processor Status, Stack Pointer, General Purpose Registers. |
| **System-level** | Process Table Entry (PCB), Page Tables, Kernel Stack. |

## Process Control Block (PCB)

Estructura en el kernel que contiene toda la info del proceso.

### En xv6 (`struct proc`):
- `state` (enum procstate)
- `xstate` (exit status)
- `pid` (Process ID)
- `parent` (puntero al proceso padre)
- `kstack` (dirección virtual del kernel stack)
- `pagetable` (tabla de páginas del usuario)
- `ofile[NOFILE]` (archivos abiertos)
- `cwd` (directorio actual)
- `name[16]` (nombre para debugging)

### En Linux (`task_struct`):
- Lista circular doblemente enlazada (task list).

## Kernel Stack

- El kernel tiene su propio stack por proceso, separado del user stack.
- ¿Por qué no usar el stack del usuario? No es seguro, el usuario podría corromperlo.
- En context switch: se restauran registros del proceso entrante, se cambia memoria user, se apunta SP al kernel stack del proceso entrante.
- La memoria del kernel NO se modifica en context switch, es compartida.

## Anticipo: Context Switch

Se cambia:
1. User-level context (memoria →Page Tables)
2. Register context (CPU → registros)
3. System-level context (estructuras del kernel → scheduler)

Hágalo rápido → ilusión de concurrencia.