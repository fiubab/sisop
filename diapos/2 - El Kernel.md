# Resumen: Clase 2 - El Kernel

## Ejecución Directa vs Limitada

| Modo | Descripción |
|------|-------------|
| **Ejecución Directa** | Programas tienen acceso completo al hardware, sin control. El SO actúa como pegamento sin aislamiento (ej: DOS). |
| **Ejecución Limitada** | Hardware limita ciertas operaciones. Un programa privilegiado (kernel) arbitra las operaciones riesgosas. |

## User Space vs Kernel Space

- **User space**: apps se ejecutan en modo usuario. No pueden ejecutar instrucciones privilegiadas.
- **Kernel space**: solo el kernel se ejecuta en modo supervisor. Puede ejecutar instrucciones privilegiadas.
- **Principio de separación mecanismo/política**: el hardware provee el mecanismo de protección; el kernel determina la política.

## Modos de Ejecución en Hardware

| Arquitectura | Modos |
|--------------|-------|
| **RISC-V** | Machine, Supervisor, User |
| **x86** | Ring 0 (supervisor), Ring 3 (usuario). También Ring 1 y 2 pero rara vez usados. |

## Instrucciones Privilegiadas

- Definición informal: toda instrucción que el modo usuario no puede ejecutar.
- Si se intenta ejecutar en modo usuario → el procesador genera una excepción.
- Ejemplos (x86): `MOV` a/desde CR0-CR4, `HLT`, `LGDT`, instrucción `OUT` (I/O).
- **Importante**: `INT n` (interrupción de software) **NO es privilegiada**, pero su manejo puede involucrar código privilegiado. Se usa para system calls.

## Protección de Memoria

- Se realiza mediante tablas de páginas.
- Cada entrada tiene flags: acceso, escritura, mapeo, ejecución.
- Acceder a página no mapeada → excepción. Escribir página de solo lectura → excepción. Acceder a página no de usuario → excepción.

## Timer Interrupts

- Casi todos los procesadores tiene un hardware timer.
- Interrumpe al procesador a intervalos regulares → transfiere control de usuario a kernel.
- Permite al kernel recuperar el control de un proceso.

## Modos de Transferencia

### De User a Kernel:
| Causa | Descripción |
|-------|-------------|
| **Interrupciones** | Señal asincrónica de dispositivos I/O o timer. Causa cambio a kernel mode. |
| **Excepciones** | Evento de hardware causado por una aplicación (ej: page fault, división por cero). |
| **System Calls** | Transición voluntaria de usuario a kernel para solicitar un servicio. |

### De Kernel a User:
- Continuar después de interrupción/excepción/syscall
- Context switch a otro proceso
- Nuevo proceso

## System Calls

- Punto de entrada controlado al kernel.
- Cambia modo de procesador de user a kernel mode.
- El conjunto de syscalls es fijo, cada una identificada por un número.
- Cada syscall tiene parámetros que se transfieren desde user space a kernel space.

### Protocolo de System Call en x86:
1. El programa llama a una función wrapper en la biblioteca de C.
2. El wrapper copia el número de syscall a `%eax`, y los argumentos a `%ebx`, `%ecx`, `%edx`.
3. El wrapper ejecuta `int 0x80` o `SYSCALL` (trap instruction).
4. El procesador cambia a kernel mode y ejecuta el handler `system_call()`.
5. El handler: guarda registros en el stack del kernel, verifica validez, invoca el servicio, devuelve resultado.
6. Se restauran registros, se agrega valor de retorno, se pasa a user mode.
7. Si hay error, el wrapper setea `errno`.

### Herramientas:
- `strace`: traza syscalls de un proceso.
- `ltrace`: traza llamadas a bibliotecas.

## Vector de Interrupciones

- Array en memoria donde cada entrada contiene la dirección de un handler y configuración de privilegio.
- Cada interrupción tiene un número asociado.
- Orden de importancia: errores de máquina > timers > discos > network > terminales > software.