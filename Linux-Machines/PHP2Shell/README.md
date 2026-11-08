# TryHackMe: PHP2Shell

Write-up de la sala [PHP2Shell](https://tryhackme.com/room/cvephp2shell), una máquina Linux de TryHackMe. Aquí se documentarán el reconocimiento, la enumeración, el acceso inicial y la escalada de privilegios, según los resultados obtenidos durante la práctica.

> **Alcance:** procedimiento realizado en la máquina asignada por TryHackMe. La IP cambia entre sesiones; reemplaza `<IP_DE_LA_MAQUINA>` por la dirección que muestra tu sala.

## Resumen

| Fase | Hallazgo |
|---|---|
| Reconocimiento | `[Completar con los puertos y servicios identificados]` |
| Enumeración | `[Completar con las rutas, servicios o usuarios relevantes]` |
| Acceso inicial | `[Completar con el vector de acceso y el usuario obtenido]` |
| Escalada | `[Completar con la técnica utilizada y el resultado]` |

## Procedimiento

### 1. Reconocimiento de servicios

`[Documentar el escaneo realizado, los puertos abiertos, las versiones detectadas y la evidencia correspondiente.]`

```bash
# Añadir aquí el comando utilizado y sustituir la IP por la de la máquina.
```

![Reconocimiento de servicios](evidencias/01-recon.png)

### 2. Enumeración

`[Documentar la enumeración de los servicios detectados y los hallazgos que orientaron los siguientes pasos.]`

![Hallazgos de enumeración](evidencias/02-enumeracion.png)

### 3. Acceso inicial

`[Describir el vector confirmado, los pasos realizados y el usuario o contexto obtenido. Evitar incluir flags en el texto o en las capturas.]`

![Acceso inicial](evidencias/03-acceso-inicial.png)

### 4. Escalada de privilegios

`[Documentar la enumeración local, el vector de escalada confirmado y cómo se verificaron los privilegios obtenidos.]`

![Escalada de privilegios](evidencias/04-escalada.png)

## Lecciones y remediación

- `[Añadir las causas técnicas de los hallazgos y las medidas de mitigación aplicables.]`
- `[Añadir recomendaciones de actualización, configuración o control de acceso que correspondan.]`

## Nota sobre las evidencias

Guardar en `evidencias/` únicamente las capturas seleccionadas para el write-up. Antes de publicar, revisar que no expongan flags, credenciales, direcciones IP u otros datos personales.

---

## ✍️ Autoría

**Patricio Allendes** · [PatoLock](https://github.com/patolock)  
*Estudiante de Ciberseguridad · Pentester Trainee*

> *Documentar para aprender; aprender para proteger.*