# Resumen: Clase 9 - File System II

## API de Archivos

### open():
- Convierte un nombre de archivo en un FD. Devuelve el menor FD disponible.
- Flags principales: O_RDONLY, O_WRONLY, O_RDWR, O_APPEND, O_CREAT, O_TRUNC.
- Permisos (con O_CREAT): S_IRWXU(0700), S_IRUSR(0400), S_IWUSR(0200), S_IXUSR(0100), S_IRWXG(0070), etc.

### creat():
- Equivale a `open()` con `O_CREAT|O_WRONLY|O_TRUNC`.

### close():
- Cierra un FD. Error si ya está cerrado.

### read():
- Lee hasta N bytes desde la posición actual del FD. Incrementa offset.

### write():
- Escribe hasta `count` bytes desde un buffer al FD. Puede escribir menos bytes de los solicitados.

### lseek():
- Reposiciona el offset de un FD:
  - `SEEK_SET`: desde el inicio.
  - `SEEK_CUR`: desde la posición actual.
  - `SEEK_END`: desde el final.

### dup() y dup2():
- `dup()`: duplica un FD, devuelve el menor número disponible.
- `dup2()`: duplica FD en un número específico (`newfd`).
- Ambos FDs comparten offset y flags de estado.

#### Tres tablas del kernel:
1. **Per-process FD table**: por proceso. Contiene flags y referencia al open file descriptor.
2. **System-wide open file table**: contiene offset actual, flags de estado, modo de acceso, referencia al i-nodo.
3. **i-node table**: por filesystem. Contiene tipo de archivo, locks, propiedades.

## Hard Links y Soft Links

| Característica | Hard Link | Soft Link (Symlink) |
|---------------|-----------|---------------------|
| Inodo apunta a | Datos del archivo | Bloque con la ruta del archivo |
| Relación | Mismo inodo | Archivo separado |
| Si se borra el destino | Sigue funcionando | Se rompe |
| Directorios | No se permite | Se permite |
| Diferentes particiones | No se permite | Se permite |
| Espacio extra | No | Sí (archivo pequeño) |

### link():
- Crea un nuevo nombre para un archivo (hard link). Ambos nombres son equivalentes.

### symlink():
- Crea un link simbólico a un archivo.

### unlink():
- Elimina un nombre del filesystem. Si era el último link y nadie tiene el archivo abierto → lo borra completamente.

## API de Directorios

### mkdir():
- Crea directorios.

### opendir(), readdir(), closedir():
- `opendir()`: abre un stream de directorio.
- `readdir()`: lee próxima entrada. Devuelve `struct dirent` con `d_name[]` y `d_ino`.
- `closedir()`: cierra el stream.

## Resolución de Paths (Ejercicios de Parcial)

### `ls -l /dir/x`:
- Lee inodo `/` (DIRECTORIO) → Lee bloque `/` → Lee inodo `dir` (DIRECTORIO) → Lee bloque `dir` → Lee inodo `x` (REGULAR)
- **3 inodos, 2 bloques** (ls no accede a datos del archivo)

### `cat /dir/s/y`:
- Lee inodo `/` → bloque `/` → inodo `dir` → bloque `dir` → inodo `s` → bloque `s` → inodo `y` (REGULAR) → bloque datos de `y`
- **4 inodos, 4 bloques**

### `cat /dir/h` (después de `ln /dir/x /dir/h` y `rm /dir/x`):
- Hard link: el inodo del archivo sigue existiendo.
- Lee inodo `/` → bloque `/` → inodo `dir` → bloque `dir` → inodo `h` (REGULAR) → bloque datos de `h`
- **3 inodos, 3 bloques**

### `cat /dir/y` (donde `/dir/y` es symlink a `/dir/s/y`):
- Lee inodo `/` → bloque `/` → inodo `dir` → bloque `dir` → inodo `y` (SYMLINK) → bloque de `y` (obtiene ruta `/dir/s/y`) → Lee inodo `/` → bloque `/` → inodo `dir` → bloque `dir` → inodo `s` → bloque `s` → inodo `y` (REGULAR) → bloque datos
- **7 inodos, 7 bloques**

## stat():
- Devuelve información sobre un archivo (metadata). No requiere permisos sobre el archivo, solo sobre los directorios del path.

## access():
- Chequea permisos: F_OK (existe), R_OK (lectura), W_OK (escritura), X_OK (ejecución).

## chmod(), chown():
- Cambian permisos y propietario/grupo del archivo.

## Implementación: Very Simple File System (vsfs)

### Requisitos de diseño:
- Estructura de índice del archivo (para localizar cada bloque).
- Mapa de espacio libre (para asignar bloques nuevos).

### Organización general:

| Región | Descripción | Tamaño típico |
|--------|-------------|---------------|
| **Superbloque (S)** | Info del FS: cant. inodos, cant. bloques, posición tabla inodos, posición bitmaps | 1 bloque |
| **Inode Bitmap (i)** | Bitmap de inodos libres/ocupados | 1 bloque |
| **Data Bitmap (d)** | Bitmap de bloques de datos libres/ocupados | 1 bloque |
| **Inode Table (I)** | Array de inodos | N bloques |
| **Data Region (D)** | Bloques de datos de archivos | Resto |

### Cálculo de tamaño (ejemplo de parcial):
- Disco de 1024 bloques, bloque 4KB, inodo 256 bytes.
- Inodos por bloque: 4096/256 = 16
- Cantidad de inodos: 1024 (criterio: 1 inodo por bloque)
- Bloques de inodos: 1024/16 = 64
- Bitmaps: 1 + 1 = 2 bloques (sobran: 4096*8 = 32768 > 1024)
- Superbloque: 1 bloque
- Bloques de datos: 1024 - 64 - 1 - 1 - 1 = 957

### Inodos:
- Referenciados por inumber (índice en la inode table).
- Para leer inodo 32: offset = 32 * sizeof(inode) + dirección_inicio_tabla.
- Contienen: metadata + punteros a bloques de datos.

### Bitmaps:
- Un bit por cada inodo/bloque: 0 = libre, 1 = ocupado.

## FFS: Fixed Tree (Unix Fast File System)

- Inodo contiene 15 punteros:
  - 12 punteros directos a bloques de datos.
  - 1 puntero indirecto (apunta a bloque con punteros directos).
  - 1 puntero indirecto doble (apunta a bloque de punteros indirectos).
  - 1 puntero indirecto triple.

### Capacidad con bloques de 4KB y punteros de 4 bytes:
- Directos: 12 bloques = 48 KB
- Indirecto: 1024 bloques ≈ 4 MB
- Doble indirecto: 1024² bloques ≈ 4 GB
- Triple indirecto: 1024³ bloques ≈ 4 TB

## FAT / FAT-32

- File Allocation Table: arreglo de entradas de 32 bits.
- Cada archivo es una lista enlazada en la FAT.
- El directory entry contiene el número del primer bloque del archivo.
- FAT[i] = 0 → bloque i libre.
- Sufre de fragmentación. Herramientas de desfragmentación reorganizan bloques secuencialmente.

## Buffer Cache / Page Cache

### Funciones del Buffer Cache:
1. Sincronizar acceso a bloques de disco (una sola copia en memoria).
2. Cachear bloques populares para evitar lectura de disco.

### Page Cache en Linux:
- Reemplazó al Buffer Cache.
- Almacena en RAM datos leídos/escritos del disco.
- Evita acceder al disco físico si los datos ya están en memoria.

### Tiempos típicos:
| Operación | Tiempo |
|-----------|--------|
| L1 cache | 0.5 ns |
| L2 cache | 7 ns |
| Mutex lock/unlock | 25 ns |
| Main memory | 100 ns |
| Read 4K random SSD | 150 µs |
| Disk seek | 10 ms |

## Primitivas de Sincronización

### fsync():
- Fuerza escritura de datos y metadata al disco.
- `fdatasync()`: solo datos, no metadata.
- `write()` solo transfiere al page cache, no garantiza persistencia en disco.

### mmap() y msync():
- `mmap()`: mapea archivo directamente en espacio de direcciones del proceso.
- `msync()`: sincroniza cambios en memoria mapeada al disco.
- Mapeos compartidos permiten IPC y I/O eficiente.
- `mmap()` por sí solo no es persistente. Usar `msync()` para garantizar persistencia.

## Clasificación de Kernels

| Tipo | Descripción | Ejemplos |
|------|-------------|----------|
| **Monolítico** | Todo corre en kernel space. Rápido pero inestable si un driver falla. | Linux, BSD |
| **Microkernel** | Mínima funcionalidad en kernel (IPC, scheduling, memoria). Lo demás en user space. Más estable, menos rendimiento. | MINIX 3, QNX |
| **Híbrido** | Un espacio de kernel pero modular. | Windows NT, macOS (XNU) |
| **Nanokernel** | Solo abstrae hardware. Para sistemas embebidos/virtualización. | - |
| **Exokernel** | No abstrae, expone hardware a aplicaciones (libOS). Experimental. | Exokernel MIT |