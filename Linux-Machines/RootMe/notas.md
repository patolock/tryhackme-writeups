# Auditoría de Máquina: [Nombre de la Máquina]

* **Plataforma:** TryHackMe
* **IP de la Máquina:** `10.10.X.X`
* **Dificultad:** [Fácil / Media / Difícil]
* **Fecha de Resolución:** 24/10/2026

---

## 1. Fase de Reconocimiento y Escaneo (Recon)

### Escaneo de Puertos con Nmap
Comando utilizado:
```bash
sudo nmap -sC -sV -oN nmap_scan.txt 10.10.X.X
```

**Puertos abiertos encontrados:**
* **Puerto 22 (SSH):** Versión OpenSSH...
* **Puerto 80 (HTTP):** Servidor Apache...
* **Puerto [Otros]:** ...

### Enumeración Web (Si aplica)
* Herramienta usada (`gobuster` / `dirsearch` / `feroxbuster`):
```bash
gobuster dir -u http://10.10.X.X/ -w /usr/share/wordlists/dirb/common.txt
```
* **Directorios interesantes encontrados:** `/admin`, `/uploads`, etc.

---

## 2. Acceso Inicial (Intrusión / Foothold)

* **Vulnerabilidad detectada:** (Ej. Inyección SQL, una contraseña por defecto, un exploit conocido para la versión del puerto X).
* **Explicación del ataque:** [Explica brevemente cómo explotaste la falla].

Comando o script utilizado:
```bash
# Aquí pegas el comando, exploit o payload que te dio acceso
python3 exploit.py 10.10.X.X
```

* **User Flag:** `THM{u_s_e_r_******}` (Recuerda censurar si el repositorio es público)

---

## 3. Escalada de Privilegios (Privilege Escalation)

### Enumeración Interna
* Comando o herramienta utilizada para buscar debilidades internas (Ej. `linpeas.sh`, buscar archivos SUID, revisar `sudo -l`):
```bash
sudo -l
```

### Explotación y Acceso Root
* **Debilidad encontrada:** (Ej. El usuario puede ejecutar un binario como root sin contraseña).
* **Proceso para escalar:** [Cómo pasaste de usuario normal a Administrador/Root].

* **Root Flag:** `THM{r_o_o_t_******}`

---

## 4. Notas y Lecciones Aprendidas
* **¿Qué aprendí de nuevo en esta máquina?:** [Ej: Una nueva bandera de una herramienta, un vector de ataque que no conocía].
* **Errores cometidos:** [Ej: Perder mucho tiempo en un puerto que era un conejo blanco / distracción].
