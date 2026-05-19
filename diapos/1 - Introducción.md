# Resumen: Clase 1 - Introducción

## ¿Qué es un Sistema Operativo?

- Capa de software que maneja los recursos de una computadora para usuarios y aplicaciones.
- Comparte hardware entre múltiples programas para que parezcan ejecutarse simultáneamente.
- Provee servicios a programas de usuario mediante una interfaz (system calls).
- Diseño de interfaz: pocos mecanismos que se combinan para dar generalidad.

## Funciones de un SO (3 metáforas)

| Metáfora | Función |
|----------|---------|
| **Referee (Árbitro)** | Gestión y distribución de recursos. Aísla aplicaciones entre sí. Protege al sistema. |
| **Illusionist (Ilusionista)** | Abstrae el hardware. Ilusión de memoria infinita y CPU exclusiva. |
| **Glue (Conector)** | Servicios comunes: compartir info entre apps, rutinas de UI, separación apps/dispositivos I/O. |

## Virtualización

- Técnica principal del SO: tomar un recurso físico y transformarlo en algo virtual más general y fácil de usar.
- Se virtualiza: CPU, Memoria, Tiempo (concurrencia), Periféricos.

## Arquitectura de Von Neumann

- Fetch-decode-execute: el procesador busca, decodifica y ejecuta instrucciones desde memoria.
- Tendencias actuales: GPUs (computación paralelo, no es Von Neumann), Cloud Computing (sistemas distribuidos).

## Unix: Filosofía y Diseño

- Creado por Ken Thompson y Dennis Ritchie (1969).
- Principios de Unix:
  1. Cada programa hace una única tarea bien. Crear programas nuevos en lugar de añadir complejidad.
  2. La salida de cada programa debe poder ser entrada de otro. Evitar formatos complejos.
  3. Diseñar para ser probado temprano. No dudar en descartar y rehacer.
  4. Usar herramientas especializadas aun si son temporales.

## Kernel y User Space

- **Kernel**: programa especial que provee servicios a procesos en ejecución. Barrera entre apps y hardware.
- **User-land**: donde viven las aplicaciones (procesos). Contexto aislado, protegido, restringido.
- Cada programa en ejecución se llama **proceso**.
- Los servicios del kernel se acceden mediante **system calls**.

## Requisitos Clave de un SO

| Requisito          | Descripción                                                                                   |
| ------------------ | --------------------------------------------------------------------------------------------- |
| **Multiplexación** | Dividir un proceso en varios que se ejecutan simultáneamente, compartiendo tiempo y recursos. |
| **Aislamiento**    | Si un proceso falla, no afecta a otros. No es absoluto: permite interacción controlada.       |
| **Interacción**    | Comunicación intencionada entre procesos (ej: pipes). IPC inter process communication.        |
## Archivos en Unix: "Casi todo es un archivo"

- API universal: `open()`, `close()`, `read()`, `write()`.
- Un **file descriptor** es un entero que el SO asigna al abrir un recurso.
- Cosas representables como FD: archivos normales, dispositivos I/O, sockets de red, pipes/FIFOs, pseudoterminales.
- Por convención, los 3 FDs estándar: stdin(0), stdout(1), stderr(2). El shell los abre en cada proceso que se crea.

## Procesos

- Un proceso es una **instancia de un programa en ejecución**. Es dinámico, tiene estructura interna.
- Componentes: Code, Data, Heap, Stack, File Descriptors.
- Todos los procesos menos el kernel viven en user-land.

## API de Procesos (System Calls)

| Syscall                        | Descripción                                                                                          |
| ------------------------------ | ---------------------------------------------------------------------------------------------------- |
| `fork()`                       | Crea copia exacta del proceso padre. Devuelve PID del hijo al padre, 0 al hijo.                      |
| `wait()`                       | Bloquea al padre hasta que un hijo termine. Devuelve PID del hijo terminado.                         |
| `getpid()`                     | Devuelve el PID del proceso actual. `getppid()` devuelve el del padre.                               |
| `execve(pathname, argv, envp)` | Reemplaza el proceso actual con un nuevo programa. No crea proceso nuevo. No cambia PID. Hereda FDs. |
| `exit()`                       | Termina el proceso actual con código de salida.                                                      |
| `kill(pid)`                    | Envía señal para terminar el proceso indicado.                                                       |
| `pipe(fd[2])`                  | Canal unidireccional. `fd[1]` para escritura, `fd[0]` para lectura.                                  |
| `dup(fd)`                      | Duplica un FD, devuelve el menor número libre.                                                       |
| `dup2(fdA, fdB)`               | Duplica fdA en fdB.                                                                                  |

### Puntos clave de fork():

- Padre e hijo son copias exactas.
- Después de fork(), ambos se ejecutan por separado.
- El orden de ejecución es indeterminado.
- El hijo hereda los FDs del padre.

### Puntos clave de execve():

- No crea nuevo proceso, solo reemplaza el contenido (código, datos, stack, heap).
- No cambia PID ni FDs.

### Pipes:

- Buffer en el kernel expuesto como par de FDs.
- Unidireccional.
- Cerrar un FD en un proceso no lo cierra en el otro.