# Resumen: Clase 8 - File System I

## ¿Qué es un File System?

- Abstracción del SO que provee **datos persistentes con un nombre**.
- Datos persistentes: se almacenan hasta que se borran explícitamente, sobreviven a cortes de energía.
- Permite compartir información entre programas.

## Decisiones de Diseño de Unix FS:
1. Estructura jerárquica
2. Tratamiento consistente de datos (streams de bytes)
3. Crear y borrar archivos
4. Crecimiento dinámico
5. Protección de datos
6. Tratamiento de periféricos como archivos

## Estructura Jerárquica
- Árbol: directorio raíz → directorios → archivos (hojas).

## Tratamiento Consistente
- Los programas ven archivos como **streams de bytes**.
- En nivel de syscalls (read/write) no se asume codificación ni formato.

## Permisos
- Esquema clásico de UNIX:
- 3 permisos: read (r), write (w), execute (x)
- 3 clases: owner, group, others
- Representación: `rwxr-xr--` → 754

## Dispositivos como Archivos
- Todo es un archivo en UNIX: discos, terminales, /proc filesystem.
- Permite acceso uniforme y manejo de permisos unificado.
- `dd if=/dev/sdc of=/dev/sdd bs=64K conv=noerror,sync` → clonar disco.

## Abstracciones del File System

### Inodo (Inode):
- Almacena **metadatos** del archivo (NO el contenido):
  - Tamaño, propietario, grupo, permisos, timestamps, cantidad de links, punteros a bloques de datos.
- Tipos de inodo en Linux: regular, directorio, dispositivo de bloque, dispositivo de carácter, link simbólico, socket, FIFO.

### Dentry (Directory Entry):
- Representa la relación entre un **nombre** y su **inodo**.
- Los directorios son listas de dentries.
- Múltiples dentries pueden apuntar al mismo inodo → **hard link**.
- Un dentry apunta a inodos, nunca a otros dentries.
- Un inodo apunta a datos, nunca a otros inodos.
- La jerarquía se forma porque un inodo tipo directorio apunta a datos que contienen listas de dentries.

### Open File Table:
- Vincula file descriptors con inodos.
- Contiene el **offset** ("cursor" de lectura/escritura).
- Tres tablas: Per-process FD table, System-wide open file table, i-node table.

## Archivo = Dentry + Inodo
- El archivo se implementa como un dentry apuntando a un inodo.

## Virtual File System (VFS)

- Subsistema del kernel que implementa la interfaz para archivos y filesystems.
- Permite coexistir e inter-operar diferentes filesystems.
- VFS es el pegamento: `open()`, `read()`, `write()` funcionan sin importar el hardware subyacente.
- Define interfaces comunes que todo filesystem debe implementar.

### Objetos del VFS:
| Objeto | Representa |
|--------|-----------|
| **Super bloque** | Un filesystem montado |
| **Inodo** | Un archivo |
| **Dentry** | Una entrada de directorio |
| **File** | Un archivo abierto por un proceso |

### Operaciones del VFS:
- `super_operations`: sobre el filesystem (write_inode, sync_fs).
- `inode_operations`: sobre un archivo (create, link).
- `dentry_operations`: sobre una dentry (d_compare, d_delete).
- `file_operations`: sobre un archivo abierto (read, write).

## Dispositivos

### /dev:
- Los dispositivos se representan como archivos especiales de tipo DEVICE.
- Se usan las mismas syscalls que para archivos normales.
- El VFS interactúa con drivers que implementan las operaciones de bajo nivel.

### Tipos:
| Tipo | Transferencia | Ejemplos |
|------|---------------|----------|
| **Dispositivos de carácter** | Byte a byte | Terminales, puertos serie |
| **Dispositivos de bloque** | Bloques de tamaño fijo | Discos, SSD, USB |

### Números Major y Minor:
- **Major**: identifica el driver que maneja el dispositivo.
- **Minor**: identifica un dispositivo específico dentro del grupo.

### mknod y mount:
```bash
sudo mknod /dev/ttyS1 c 4 65    # Crear nodo de dispositivo
sudo mount -t ext4 /dev/sda1 /mnt/dir  # Montar filesystem
```

### Filesystems virtuales:
- No corresponden a un dispositivo físico. Ejemplo: procfs.
- `mount -t proc proc /proc` → provee info de procesos y datos del kernel.
- Otros: sysfs, tmpfs, cgroup