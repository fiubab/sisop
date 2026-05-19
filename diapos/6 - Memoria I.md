# Resumen: Clase 6 - Memoria I

## Introducción Histórica

- **Early Days**: SO en 64KB, resto disponible para un único proceso.
- **Multiprogramación**: múltiples procesos listos, SO intercala ejecución.
- **Time Sharing**: compartir recurso computacional concurrentemente mediante multiprogramación e interrupciones de reloj.

## Address Space (Espacio de Direcciones)

- El Address Space de un proceso contiene todo el estado de memoria del programa en ejecución.
- Segmentos: Text (código), Data (globales), Heap (dinámica), Stack (locales).

## Virtualización de Memoria

Metas:
1. **Transparencia**: el programa no debe saber que la memoria es virtual.
2. **Eficiencia**: no hacer más lento el programa, no usar demasiada memoria para estructuras.
3. **Protección**: un proceso no puede acceder a la memoria de otro ni del SO.

## Address Translation

- El hardware transforma direcciones virtuales en físicas mediante la MMU.
- Formalmente: MAP: VAS → PAS ∪ ∅
- El SO debe: configurar el hardware, gestionar memoria libre/ocupada, mantener control sobre el uso.

## Implementaciones de Memoria Virtual

### Base and Bound (Segmentación Simple)

- Dos registros hardware por CPU: registro base y registro límite (bound).
- Dirección física = dirección virtual + registro base.
- Si dirección virtual > registro bound → excepción (segmentation fault).
- Problema: un solo segmento, fragmentación interna.

### Segmentación con Tabla de Segmentos

- Múltiples pares (base, bound) por proceso.
- Dirección virtual = número de segmento : offset.
- El número de segmento es índice en la tabla.
- Los bits de mayor orden seleccionan el segmento; el resto es offset.
- Ventaja: múltiples segmentos con diferentes permisos.
- Desventaja: fragmentación externa.

### Memoria Paginada

- La memoria se divide en **páginas** (tamaño fijo, potencia de 2, ej: 4KB).
- Address space se divide en páginas virtuales → se mapean a frames físicos.
- **Page Table**: por cada proceso, mapea páginas virtuales a frames físicos.
- No hay fragmentación externa, solo interna (última página puede desperdiciar espacio).
- Dirección virtual = número de página virtual : offset.
- Dirección física = frame_number (desde page table) : offset.

### Memoria Paginada Multinivel

- Se agregan niveles de tablas de páginas para reducir el tamaño de la page table en memoria.
- x86 (32 bits): 2 niveles → Page Directory (1024 entradas) → Page Table (1024 entradas) → Frame (4KB).
- RISC-V (64 bits): 3 niveles (en xv6), el satp indica qué page table usar.

## Casos de Estudio

### Base and Bound en 8086:
- Bus de datos de 20 bits, registros de 16 bits.
- Dirección física = segment_register << 4 + offset.
- Combinaciones: CS:IP (código), SS:SP (stack), DS:BX/DI/SI (datos), ES:DI (extra data).

### Modo Protegido x86 (80286+):
- Tabla de segmentos con descriptores de 8 bytes.
- GDT (Global Descriptor Table): una por sistema, apuntada por GDTR.
- LDT (Local Descriptor Table): una por tarea.
- Segment Selector: índice (13 bits) + TI + RPL.
- Segment Descriptor: base (32 bits), límite (20 bits), tipo, DPL, flags.

### Segmentación en Linux:
- Linux prefiere paginación a segmentación (más simple, más portable).
- Todos los segmentos comienzan en 0x00000000, haciendo que direcciones lógicas = direcciones lineales.

## Memoria Paginada en x86

### Page Directory Entry (PDE):
- Present (P), Read/Write (R/W), User/Supervisor (U/S), Write-through (W/T), Cache Disable (C), Accessed (A), Dirty (D), Page Size (PS), Global (G).

### Page Table Entry (PTE):
- Similar a PDE pero a nivel de página. Bits importantes: P, R/W, U/S, A, D, G.

### Registros especiales:
- **CR0**: bit PE (modo protegido), bit PG (paginación habilitada), bit WP (protección de escritura).
- **CR3**: puntero al Page Directory. Al cambiar CR3 → se invalida TLB → se cambia el address space.

### Ejemplo de traducción (32 bits, páginas de 4KB):
- Dirección virtual: `0x01FBD000 = 0000000111 1110111101 000000000000`
- Índice PDE: 7, Índice PTE: 957, Offset: 0.
- Se accede: PDE[7] → PTE[957] → Frame físico + offset.

### Ejercicio de parcial:
- Array de 50,000 enteros (4 bytes c/u = 200,000 bytes) empezando en `0x01FBD000`.
- Rango: `[0x01FBD000, 0x01FEDD3F]` → PDE[7], PTE desde 957 hasta 1005.
- Resultado: 1 PDE + 1 PTE + 49 frames de datos = **51 frames**.

## Translation Lookaside Buffer (TLB)

- Cache de traducciones virtuales → físicas dentro de la MMU.
- TLB hit: traducción rápida sin consultar page table.
- TLB miss: hay que recorrer la page table (costoso).
- Implementado en memoria estática on-chip, muy rápida.
- Puede tener múltiples niveles (L1 TLB, L2 TLB).

### Tipos de TLB:
- **Full-associative**: se busca en paralelo en todas las entradas.
- **Set-associative**: cada page number se guarda en posición definida por bits de la dirección.

### Consistencia de la TLB:
1. **Context switch**: las direcciones virtuales del proceso viejo no son válidas para el nuevo → se hace flush de TLB o se taguean entradas con PID del proceso (ASID).
2. **Reducción de permiso**: cuando el SO modifica una page table entry, debe invalidar la entrada correspondiente en la TLB.
3. **TLB shutdown**: en multiprocesador, al modificar una PTE, hay que invalidar esa entrada en todas las TLB de todos los CPUs. Se manda interrupción a cada CPU para que invalide su entrada. Operación muy costosa.