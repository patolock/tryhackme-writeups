# Análisis de vulnerabilidades y recomendaciones — TryHackMe RootMe

## 1. Resumen ejecutivo

Las capturas seleccionadas muestran dos hallazgos encadenados: se acepta un archivo desde `/panel/`, aparece contenido cargado en `/uploads/` y se recibe una shell como `www-data`; además, la enumeración muestra `/usr/bin/python2.7` entre los binarios SUID y otra captura confirma una shell como `root`.

| ID | Hallazgo | CWE | CVE aplicable | CVSS v3.1 estimado | Criticidad |
|---|---|---|---|---:|---|
| V-01 | Carga web seguida de shell como `www-data` | [CWE-434](https://cwe.mitre.org/data/definitions/434.html) | No identificado para esta configuración | 9.8* | **Crítica*** |
| V-02 | Intérprete Python 2.7 con permiso SUID | [CWE-250](https://cwe.mitre.org/data/definitions/250.html) | No identificado para esta configuración | 7.8 | **Alta** |

Las puntuaciones son estimaciones con CVSS v3.1, no puntuaciones oficiales asignadas por NIST/NVD ni por el proveedor. **V-01 es provisional**: su vector supone que no se requieren privilegios previos ni interacción adicional (`PR:N/UI:N`), aspectos que no quedan determinados por las capturas. Si se confirma autenticación u otras condiciones distintas, se debe recalcular la puntuación. El compromiso completo resulta de encadenar V-01 y V-02; las puntuaciones individuales no se suman.

## 2. Alcance, método y limitaciones

- **Alcance:** máquina de laboratorio RootMe asignada por TryHackMe y evidencias finales guardadas en [`evidencias/`](evidencias/).
- **Método:** revisión de las capturas seleccionadas: escaneo de los puertos 22 y 80, enumeración de `/panel/` y `/uploads/`, recepción de una shell como `www-data`, búsqueda de archivos SUID y verificación de identidad `root`.
- **Criterio de severidad:** rangos cualitativos de CVSS v3.1: Crítica (9.0–10.0), Alta (7.0–8.9), Media (4.0–6.9), Baja (0.1–3.9) y Ninguna (0.0). La puntuación Base no sustituye un análisis de riesgo del entorno real.
- **Limitación:** se analizaron las evidencias disponibles; no se realizó una auditoría independiente del código fuente, de la configuración completa del servidor ni de otros hosts.

## 3. Hallazgos

### V-01 — Carga web y shell remota como `www-data`

**Criticidad estimada:** Crítica* — **CVSS v3.1: 9.8***  
**Vector:** `CVSS:3.1/AV:N/AC:L/PR:N/UI:N/S:U/C:H/I:H/A:H`  
**CWE:** [CWE-434 — Unrestricted Upload of File with Dangerous Type](https://cwe.mitre.org/data/definitions/434.html)  
**CVE:** No se identificó un CVE aplicable directamente al formulario y a la configuración observados.

#### Evidencia y análisis

La captura del panel `/panel/` muestra el mensaje de carga exitosa. Otra captura muestra archivos con extensiones PHP en `/uploads/`, y una terminal registra una conexión entrante con el usuario `www-data`. En conjunto, esto es consistente con la ejecución de un archivo cargado; las capturas no muestran el contenido del archivo ni permiten determinar por sí solas todas las validaciones del servidor.

`*` Estimación provisional, condicionada a los supuestos `PR:N/UI:N` del vector. Las capturas no permiten confirmar esos requisitos ni derivar una puntuación definitiva.

- [Confirmación de la carga](evidencias/03-carga-archivo.png)
- [Shell recibida como `www-data`](evidencias/04-shell-www-data.png)

La puntuación CVSS indicada es provisional: la evidencia muestra acceso al panel y una conexión de retorno, pero no documenta de forma suficiente los requisitos de autenticación ni el impacto independiente de esta falla. El vector refleja un escenario de laboratorio supuesto y debe recalcularse si esos requisitos difieren en el sistema evaluado.

#### Recomendaciones de mitigación

1. **Impedir la ejecución en el directorio de cargas.** Configurar el servidor web para que `/uploads/` no procese PHP ni ningún otro tipo de script.
2. **Almacenar fuera del webroot.** Servir los archivos a través de un controlador que aplique autorización y tipo de contenido, o desde un host separado sin intérpretes ejecutables.
3. **Aplicar una lista permitida mínima.** Aceptar únicamente extensiones y formatos requeridos por el negocio; no depender de una lista de extensiones bloqueadas.
4. **Validar el contenido en el servidor.** No confiar en el nombre ni en el encabezado `Content-Type` enviado por el cliente. Comprobar la firma y el formato esperado, y rechazar discordancias.
5. **Generar nombres de almacenamiento aleatorios** y limitar tamaño, cantidad y permisos de los archivos.
6. **Revisar y retirar cargas existentes** que puedan ejecutarse; verificar registros HTTP para detectar accesos a archivos sospechosos.

Como guía general de mitigación, OWASP recomienda listas permitidas, validación del contenido y almacenamiento fuera del webroot; también advierte que bloquear extensiones por sí solo es débil.

### V-02 — Python 2.7 ejecutable con el bit SUID

**Criticidad:** Alta — **CVSS v3.1: 7.8**  
**Vector:** `CVSS:3.1/AV:L/AC:L/PR:L/UI:N/S:U/C:H/I:H/A:H`  
**CWE:** [CWE-250 — Execution with Unnecessary Privileges](https://cwe.mitre.org/data/definitions/250.html)  
**CVE:** No se identificó un CVE aplicable directamente a este permiso SUID del sistema.

#### Evidencia y análisis

La enumeración de archivos con SUID incluye `/usr/bin/python2.7`. La evidencia posterior muestra la invocación del intérprete y confirma `root` mediante `whoami`.

- [Enumeración SUID, con la flag excluida del recorte](evidencias/05-enumeracion-suid.png)
- [Verificación de identidad `root`](evidencias/06-root-verificado.png)

El vector requiere acceso local previo a una cuenta sin privilegios —en este caso, la shell obtenida como `www-data`—; una vez disponible, la escalada no requiere interacción de otra persona. El intérprete permite ejecutar operaciones arbitrarias, por lo que la configuración anula la separación de privilegios del sistema.

#### Recomendaciones de mitigación

1. **Retirar el bit SUID de Python** si no existe una necesidad técnica aprobada. Los intérpretes interactivos no deben ejecutarse con privilegios elevados.
2. **Auditar todos los archivos SUID/SGID** y mantener una lista autorizada; investigar cualquier binario que no forme parte de la configuración base aprobada.
3. **Aplicar mínimo privilegio.** Si una operación requiere privilegios, implementar un componente reducido y específico con permisos estrictamente limitados en lugar de elevar un intérprete general.
4. **Actualizar o retirar Python 2.7** de acuerdo con la política de soporte del sistema operativo; su antigüedad aumenta el riesgo de mantenimiento, aunque esta observación no se puntúa como CVE en este hallazgo.
5. **Verificar la remediación** comprobando que el archivo ya no tenga SUID y que la cuenta `www-data` no pueda ejecutar una ruta equivalente para obtener privilegios elevados.

### Hallazgos no confirmados

Las capturas muestran SSH en el puerto 22 y HTTP en el 80, además de identificadores de versión para OpenSSH y Apache. Un puerto abierto o una versión visible no demuestra por sí solo una vulnerabilidad. No se asigna un CVE a estos servicios porque las evidencias disponibles no demuestran una condición explotable ni confirman el inventario de paquetes instalado.

## 4. Priorización de remediación

1. **V-01 — Inmediata:** deshabilitar la ejecución de scripts en `/uploads/` y aislar las cargas del webroot. Esta medida corta el vector remoto inicial.
2. **V-02 — Inmediata:** eliminar el SUID de Python y auditar los demás binarios especiales. Esto impide la elevación local observada.
3. **Validación:** repetir las pruebas dentro de un entorno autorizado, confirmar que una carga no ejecutable sigue pudiéndose manejar de forma segura y verificar que `www-data` no puede obtener privilegios de `root`.

## 5. Referencias

1. [FIRST — CVSS v3.1 Specification Document](https://www.first.org/cvss/v3.1/specification-document): definición de métricas, vectores y puntuación CVSS.
2. [NIST NVD — CVSS](https://nvd.nist.gov/vuln-metrics/cvss): información del NVD sobre CVSS y severidad.
3. [OWASP — File Upload Cheat Sheet](https://cheatsheetseries.owasp.org/cheatsheets/File_Upload_Cheat_Sheet.html): validación, listas permitidas y almacenamiento seguro de archivos.
4. [MITRE CWE-434](https://cwe.mitre.org/data/definitions/434.html): carga de archivos de tipo peligroso.
5. [MITRE CWE-250](https://cwe.mitre.org/data/definitions/250.html): ejecución con privilegios innecesarios.

---

## ✍️ Autoría

**Patricio Allendes** · [PatoLock](https://github.com/patolock)  
*Estudiante de Ciberseguridad · Pentester Trainee*

> *Documentar para aprender; aprender para proteger.*
