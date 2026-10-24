# 💀 TryHackMe: RootMe

**RootMe** es una máquina de nivel fácil diseñada para practicar habilidades esenciales de pruebas de penetración (pentesting). El desafío simula un escenario real donde un servidor web mal configurado permite el compromiso total del sistema operativo subyacente.

---

## 🎯 Objetivos de la Auditoría
*   [ ] **Reconocimiento:** Descubrir puertos abiertos y servicios expuestos.
*   [ ] **Enumeración Web:** Localizar directorios ocultos en el servidor HTTP.
*   [ ] **Acceso Inicial:** Evadir filtros de subida de archivos para ejecutar una *Reverse Shell*.
*   [ ] **Escalada de Privilegios:** Explotar un binario con permisos SUID inusuales para convertirse en `root`.

---

## 🛠️ Stack Tecnológico Utilizado
*   **Escaneo:** `nmap`
*   **Fuzzing Web:** `gobuster` / Wordlist: `common.txt`
*   **Explotación:** *PHP Reverse Shell* (Modificada a `.phtml`)
*   **Intercepción:** `netcat` (`nc`)
*   **Post-Explotación:** Referencias de `GTFOBins` (Python SUID)

---

## 🚀 Metodología Paso a Paso

### 1. Reconocimiento (Recon)
Se inicia con un escaneo agresivo de puertos para detectar qué ventanas de entrada tiene el servidor:
```bash
sudo nmap -sC -sV -oN nmap_scan.txt <IP_DE_LA_MAQUINA>
```
*   **Resultados:** Se identifican el puerto `22` (SSH) y el puerto `80` (HTTP corriendo Apache 2.4.29).

### 2. Enumeración de Directorios Web
Al investigar el puerto 80, no se observan enlaces directos interesantes en la página de inicio. Se ejecuta un ataque de fuerza bruta a las rutas URL utilizando un diccionario de palabras común:
```bash
gobuster dir -u http://<IP_DE_LA_MAQUINA>/ -w /usr/share/wordlists/dirb/common.txt
```
*   **Rutas críticas expuestas:**
    *   `/panel/`: Un portal web diseñado para cargar archivos.
    *   `/uploads/`: El almacenamiento público donde se guardan las cargas.

### 3. Intrusión (Foothold)
El panel de carga bloquea los archivos con extensión `.php`. Para evadir esta restricción basada en nombres de archivo (lista negra), se cambia la extensión del código malicioso a una extensión alternativa válida para el servidor de aplicaciones:
```bash
mv php-reverse-shell.php shell.phtml
```
Se sube el archivo con éxito, se activa un escucha local en nuestra máquina de ataque (`nc -lvnp 1234`) y se visita la ruta `http://<IP_DE_LA_MAQUINA>/uploads/shell.phtml` para disparar la ejecución del script.

*   **Flag de Usuario:** Ubicada en `/var/www/user.txt` -> `THM{y0u_g0t_a_sh3ll_******}`

### 4. Escalada de Privilegios (PrivEsc)
Una vez dentro del sistema como el usuario web restringido (`www-data`), se buscan binarios del sistema que tengan configurado el bit **SUID** (permiten ejecutarse con los privilegios del creador del archivo, en este caso, root):
```bash
find / -perm -u=s -type f 2>/dev/null
```
El escaneo revela que **`/usr/bin/python`** posee el bit SUID activo. Utilizando una línea de comandos específica para romper entornos restringidos (técnica documentada en GTFOBins), se invoca una consola del sistema preservando los privilegios superiores:
```bash
/usr/bin/python -c 'import os; os.execl("/bin/sh", "sh", "-p")'
```

*   **Flag de Root:** Ubicada en `/root/root.txt` -> `THM{pr1v1l3g3_3sc4l4t10n_******}`

---

## 📝 Conclusiones y Remediación
1.  **Filtros de Subida Débiles:** Las restricciones basadas únicamente en extensiones de archivos (listas negras) son ineficaces si no bloquean todas las variantes de ejecución de scripts web (como `.php5`, `.phtml`, `.phar`). Se debe implementar una lista blanca estricta y renombrar los archivos subidos de forma aleatoria.
2.  **Permisos SUID Peligrosos:** Asignar atributos SUID a intérpretes interactivos de comandos enteros (`python`, `perl`, `bash`, `awk`) introduce un riesgo crítico inmediato que invalida la separación de privilegios del sistema operativo.

---

## ✒️ Autoría y Certificación
*   **Auditor de Seguridad:** Patricio Allendes / PatoLock
*   **Contacto / Portafolio:** https://github.com/patolock
*   **Rol:** Estudiante de Ciberseguridad / Pentester Trainee
