# Kali Linux — Día 1
> Apuntes de la primera sesión: navegación, permisos, redes y crackeo de contraseñas

---

## Tabla de contenidos
1. [Navegación básica](#navegación-básica)
2. [Archivos y directorios](#archivos-y-directorios)
3. [Permisos](#permisos)
4. [Usuarios y privilegios](#usuarios-y-privilegios)
5. [Redes con nmap](#redes-con-nmap)
6. [Wordlists](#wordlists)
7. [Crackeo de contraseñas](#crackeo-de-contraseñas)
8. [Hashes — teoría](#hashes--teoría)

---

## Navegación básica

```bash
pwd                  # Muestra en qué directorio estás
ls                   # Lista archivos y carpetas
ls -la               # Lista con detalles: permisos, tamaño, fecha, archivos ocultos
cd /ruta             # Moverse a una ruta absoluta (desde la raíz /)
cd carpeta           # Moverse a una ruta relativa (desde donde estás)
cd ~                 # Ir al home del usuario actual (/home/kali)
cd /                 # Ir a la raíz del sistema
clear                # Limpiar la terminal (también Ctrl+L)
```

### Rutas absolutas vs relativas
- **Absoluta**: empieza con `/` — siempre desde la raíz. Ej: `/home/kali/hacking`
- **Relativa**: sin `/` al inicio — desde donde estás. Ej: `cd hacking`
- **`~`** es un shortcut a `/home/kali` (o el home del usuario activo)

### Carpetas importantes de la raíz `/`

| Carpeta | Para qué sirve |
|---|---|
| `/bin` | Comandos básicos del sistema |
| `/etc` | Archivos de configuración |
| `/home` | Carpetas personales de usuarios |
| `/root` | Home del usuario root |
| `/tmp` | Archivos temporales (se borra al reiniciar) |
| `/var/log` | Logs del sistema |
| `/proc` | Info en tiempo real de CPU, RAM, procesos |
| `/usr` | Programas instalados |
| `/dev` | Dispositivos (discos, USB, etc.) |

---

## Archivos y directorios

```bash
mkdir nombre         # Crear carpeta
touch archivo.txt    # Crear archivo vacío
echo "texto" > archivo.txt    # Escribir en archivo (sobreescribe)
echo "texto" >> archivo.txt   # Agregar al final del archivo
cat archivo.txt      # Leer archivo
cp origen destino    # Copiar archivo
mv origen destino    # Mover o renombrar archivo
rm archivo.txt       # Borrar archivo
rm -rf carpeta       # Borrar carpeta completa (sin papelera, irreversible)
wc -l archivo.txt    # Contar líneas de un archivo
head -20 archivo.txt # Ver primeras 20 líneas
tail -20 archivo.txt # Ver últimas 20 líneas
```

> ⚠️ `rm -rf` no tiene papelera de reciclaje. Lo borrado no se recupera.

### El pipe `|`
Toma el output de un comando y lo pasa como input al siguiente:
```bash
cat /etc/passwd | grep /bin/bash        # Filtrar usuarios con bash
cat /etc/passwd | grep -E "/bin/bash|/bin/zsh"  # Filtrar bash o zsh
```

### Redirección
```bash
>   # Sobreescribe el archivo
>>  # Agrega al final sin borrar
```

---

## Permisos

### Columnas de `ls -la`
```
drwxr-xr-x  2  kali  kali  4096  Mar 7  14:28  carpeta
│            │  │     │     │     │              │
│            │  │     │     │     └── Nombre
│            │  │     │     └── Tamaño en bytes
│            │  │     └── Grupo dueño
│            │  └── Usuario dueño
│            └── Número de enlaces
└── Permisos
```

### Tipos de archivo (primer carácter)
| Letra | Tipo |
|---|---|
| `-` | Archivo normal |
| `d` | Directorio |
| `l` | Link simbólico (como acceso directo) |
| `b` | Dispositivo de bloque (discos) |
| `c` | Dispositivo de caracteres |

### Estructura de permisos
```
drwxr-xr-x
│ │   │   │
│ │   │   └── Otros: r-x (leer y ejecutar)
│ │   └────── Grupo: r-x (leer y ejecutar)
│ └────────── Dueño: rwx (todo)
└──────────── Tipo: d = directorio
```

| Letra | Permiso | Valor numérico |
|---|---|---|
| `r` | Leer | 4 |
| `w` | Escribir | 2 |
| `x` | Ejecutar | 1 |
| `-` | Sin permiso | 0 |

### chmod — cambiar permisos
```bash
chmod 000 archivo    # Sin permisos para nadie
chmod 644 archivo    # Dueño: rw- | Grupo: r-- | Otros: r--  (normal para archivos)
chmod 755 archivo    # Dueño: rwx | Grupo: r-x | Otros: r-x  (normal para carpetas)
chmod 777 archivo    # Todos pueden todo (peligroso)
```

> **En hacking**: archivos con `777` son vulnerabilidades — cualquier usuario puede modificarlos.

> **Root** ignora todos los permisos — puede leer y escribir cualquier archivo sin importar la configuración.

---

## Usuarios y privilegios

```bash
whoami               # Ver usuario actual
id                   # Ver usuario, UID y grupos
cat /etc/passwd      # Lista de todos los usuarios del sistema
cat /etc/shadow      # Contraseñas hasheadas (requiere sudo)
sudo comando         # Ejecutar como root
sudo whoami          # Devuelve "root"
```

### Formato de `/etc/passwd`
```
usuario:x:UID:GID:descripcion:home:shell
kali:x:1000:1000::/home/kali:/usr/zsh
```

- Usuarios con `/bin/bash` o `/bin/zsh` al final = usuarios reales con acceso a terminal
- Usuarios con `/nologin` o `/false` = servicios del sistema, no pueden loguearse

### Formato de `/etc/shadow`
```
kali:$y$j9T$hash...:20519:0:99999:7:::
```
- `$y$` = algoritmo yescrypt (moderno y seguro)
- `!` o `*` en el campo de contraseña = cuenta bloqueada

### Filtrar usuarios reales
```bash
cat /etc/passwd | grep -E "/bin/bash|/bin/zsh"
```

### bash vs zsh
| | bash | zsh |
|---|---|---|
| Archivo config | `~/.bashrc` | `~/.zshrc` |
| Común en | Servidores | Kali, Mac |
| Scripts | `#!/bin/bash` | `#!/bin/zsh` |

> Kali usa **zsh** por defecto. La mayoría de servidores usan **bash**.

---

## Redes con nmap

### Ver configuración de red
```bash
ip a                 # Ver interfaces de red e IPs
```

Términos importantes:
- **eth0** — interfaz de red principal
- **inet** — dirección IPv4
- **inet6** — dirección IPv6
- **MAC address** — identificador físico de la tarjeta de red

### Comandos nmap
```bash
nmap 10.0.2.0/24           # Escanear toda la red (/24 = del .1 al .254)
nmap -sV -sC 10.0.2.2      # Detectar versiones y correr scripts básicos
nmap -O 10.0.2.2           # Detectar sistema operativo
nmap -sS 10.0.2.2          # SYN scan (silencioso, difícil de detectar)
nmap -sU 10.0.2.3          # Escaneo UDP
nmap -A 10.0.2.2           # Agresivo: OS + versiones + scripts + traceroute
nmap -p- 10.0.2.2          # Escanear los 65535 puertos (lento)
nmap -sV 10.0.2.2 -oN resultado.txt  # Guardar resultado en archivo
```

### Puertos importantes
| Puerto | Servicio |
|---|---|
| `22` | SSH |
| `80` | HTTP |
| `443` | HTTPS |
| `21` | FTP |
| `53` | DNS |
| `445` | SMB (famoso por vulnerabilidades) |
| `3306` | MySQL |
| `3389` | RDP (escritorio remoto Windows) |

> Por defecto nmap escanea los **1000 puertos más comunes** de 65535 totales.

> **Puerto 445 (SMB)**: usado en ataques históricos como EternalBlue/WannaCry.

---

## Wordlists

Ubicación en Kali: `/usr/share/wordlists/`

| Archivo/Carpeta | Para qué sirve |
|---|---|
| `rockyou.txt` | 14,344,392 contraseñas reales filtradas — la más usada |
| `dirb/` | Rutas web para directory brute force |
| `dirbuster/` | Rutas web más extensas |
| `metasploit/` | Payloads y exploits |
| `sqlmap/` | Inyecciones SQL |
| `wfuzz/` | Fuzzing web |
| `john.lst` | Contraseñas para John the Ripper |

### Descomprimir rockyou
```bash
sudo gunzip /usr/share/wordlists/rockyou.txt.gz
```

> **rockyou.txt** viene de una filtración real del 2009 — 14 millones de contraseñas en texto plano.

---

## Crackeo de contraseñas

### Generar hashes para practicar
```bash
echo -n "password123" | md5sum    # El -n evita agregar salto de línea
```
> ⚠️ Siempre usar `-n` — sin él se hashea la contraseña + un salto de línea invisible, generando un hash diferente.

### John the Ripper
```bash
john --format=raw-md5 --wordlist=/usr/share/wordlists/rockyou.txt hash.txt
john --format=raw-md5 --show hash.txt    # Ver resultado crackeado
john --format=bcrypt --wordlist=/usr/share/wordlists/rockyou.txt hash.txt
```

### Hashcat
```bash
hashcat -m 0 hash.txt /usr/share/wordlists/rockyou.txt --force    # MD5
hashcat -m 3200 hash.txt /usr/share/wordlists/rockyou.txt --force # bcrypt
```

Modos `-m` comunes:
| Número | Algoritmo |
|---|---|
| `0` | MD5 |
| `100` | SHA1 |
| `1400` | SHA256 |
| `3200` | bcrypt |
| `1800` | SHA512crypt |

### Identificar tipo de hash
```bash
hashid 482c811da5d5b4bc6d497ffa98491e38
hash-identifier    # Interactivo
nth --text "hash"  # name-that-hash, más moderno
```

---

## Hashes — teoría

### Qué es un hash
Función matemática de **una sola vía**:
```
"password123"  →  [md5]  →  "482c811da5d5b4bc6d497ffa98491e38"
```
- ✅ Contraseña → Hash: fácil
- ❌ Hash → Contraseña: matemáticamente imposible revertir

**Propiedad clave**: el mismo input siempre genera el mismo output, en cualquier máquina, en cualquier momento.

### Cómo lo usan los sistemas
```
1. Usuario escribe: "password123"
2. Sistema calcula: md5("password123") = 482c811da...
3. Compara con BD:                       482c811da...
4. ✅ Coinciden → acceso permitido
```
El sistema nunca guarda la contraseña real — solo el hash.

### Cómo funciona el crackeo (por dentro)
```python
for contraseña in rockyou:
    if md5(contraseña) == hash_objetivo:
        print("encontrada:", contraseña)
        break
```
John y Hashcat son básicamente ese loop, optimizado al máximo.

### Identificar hashes por prefijo `$`
| Prefijo | Algoritmo |
|---|---|
| `$1$` | MD5crypt |
| `$2b$` | bcrypt |
| `$5$` | SHA256crypt |
| `$6$` | SHA512crypt |
| `$y$` | yescrypt (usado en Kali) |

### MD5 vs bcrypt — velocidad de crackeo
| Algoritmo | Hashes/segundo | Tiempo para rockyou.txt |
|---|---|---|
| MD5 | ~153,600/s | **segundos** |
| bcrypt | ~150/s | **meses o años** |

bcrypt es lento **a propósito** — su work factor (`$2b$12$`) se puede aumentar para hacerlo más lento con el tiempo.

### Modos de ataque
| Modo | Cómo funciona |
|---|---|
| **Diccionario** | Itera una wordlist |
| **Fuerza bruta** | Prueba todas las combinaciones posibles |
| **Híbrido** | Wordlist + mutaciones (`password123` → `P@ssword123`) |
| **Rainbow tables** | Hashes precalculados — rápido pero ocupa mucho espacio |

---

## Herramientas vistas
| Herramienta | Para qué sirve |
|---|---|
| `nmap` | Escaneo de redes y puertos |
| `john` | Crackeo de contraseñas (CPU) |
| `hashcat` | Crackeo de contraseñas (GPU, más rápido) |
| `hashid` | Identificar tipo de hash |
| `hash-identifier` | Identificar tipo de hash (interactivo) |
| `dirb` | Directory brute force en webs |
| `gobuster` | Directory brute force (más rápido que dirb) |
| `ffuf` | Fuzzing web (el más flexible) |
| `nikto` | Escaneo de vulnerabilidades web |

---

## Próxima sesión
- Metasploit — framework de explotación
- Burpsuite — interceptar tráfico web
- TryHackMe — práctica en máquinas vulnerables reales
