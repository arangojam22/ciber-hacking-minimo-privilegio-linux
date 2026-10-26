# Actividad de ciberseguridad: auditoría con auditd en Kali Linux

Práctica de ASIR realizada en una máquina virtual de Kali Linux para vigilar modificaciones en archivos críticos. El objetivo es registrar cambios relacionados con privilegios y mantener una configuración persistente.

## 1. Preparación del laboratorio
Partí del escenario preparado por el script de la actividad, con un snapshot previo como medida de recuperación. El enunciado explica qué rutas debo vigilar y cómo comprobarlo. Esta captura documenta la preparación, no la solución.

![Preparación del escenario](<capturas/reto-08/Captura de pantalla 2026-10-26 163028.png>)

## 2. Comprobación del servicio
Ejecuté `sudo systemctl enable --now auditd` y consulté `sudo systemctl status auditd`. La salida muestra el servicio habilitado y `active (running)`. Esto confirma que el demonio está funcionando, pero no demuestra por sí solo que estén vigiladas todas las rutas necesarias. La imagen pertenece a una repetición de la práctica y ya muestra algunas reglas en la parte superior.

![Servicio auditd habilitado y activo](<capturas/reto-08/Captura de pantalla 2026-10-26 161849.png>)

## 3. Observación de la falta de vigilancia
Con `sudo auditctl -l` obtuve `No rules`. Después creé y eliminé `/etc/cron.d/prueba` para intentar localizar el cambio con `sudo ausearch -k cron`.

La captura muestra la ausencia de reglas y los comandos de prueba, pero queda cortada antes de la respuesta de ausearch: no acredita por sí sola un resultado `no matches`. Además, al repetir la práctica pueden existir eventos antiguos, que deben distinguirse por su hora.

![Ausencia de reglas y prueba inicial](<capturas/reto-08/Captura de pantalla 2026-10-26 163357.png>)

## 4. Creación de las reglas persistentes
Preparé `/etc/audit/rules.d/minpriv.rules` para vigilar estas zonas:

| Clave | Rutas | Finalidad |
|---|---|---|
| sudoers | `/etc/sudoers` y `/etc/sudoers.d/` | Detectar cambios en permisos de sudo. |
| identidad | `/etc/passwd`, `/etc/group` y `/etc/shadow` | Detectar modificaciones en cuentas y grupos. |
| cron | `/etc/cron.d/` | Detectar cambios en tareas programadas. |
| systemd | `/etc/systemd/system/` | Detectar cambios en unidades de servicios. |

Cada línea usa `-w` para la ruta, `-p wa` para escritura y cambios de atributos, y `-k` para la clave de búsqueda. El contenido correcto es:

```text
-w /etc/sudoers -p wa -k sudoers
-w /etc/sudoers.d/ -p wa -k sudoers
-w /etc/passwd -p wa -k identidad
-w /etc/group -p wa -k identidad
-w /etc/shadow -p wa -k identidad
-w /etc/cron.d/ -p wa -k cron
-w /etc/systemd/system/ -p wa -k systemd
```

Capturas del proceso de configuración. La colocación de estas dos imágenes es provisional: debe revisarse que sus salidas correspondan a este paso. Si contienen un intento previo o un error, no deben presentarse como la configuración definitiva.

![Proceso de configuración, evidencia provisional 1](<capturas/reto-08/Captura de pantalla 2026-10-26 171848.png>)

![Proceso de configuración, evidencia provisional 2](<capturas/reto-08/Captura de pantalla 2026-10-26 171939.png>)

## 5. Carga de las reglas y corrección de errores
Cargué la configuración con `sudo augenrules --load` y revisé las reglas mediante `sudo auditctl -l`.

Durante la práctica escribí incorrectamente la clave de systemd. Una regla cargada con una etiqueta distinta no satisface la comprobación que busca `systemd`. Corregí la última línea a `-w /etc/systemd/system/ -p wa -k systemd`, guardé el fichero y volví a cargarlo.

Esta captura documenta un intento pendiente de corrección, no un resultado final satisfactorio:

![Intento con una clave de systemd incorrecta](<capturas/reto-08/Captura de pantalla 2026-10-26 172542.png>)

Después de corregirla, la lista muestra las siete reglas y la clave `systemd` correctamente escrita:

![Reglas activas con las claves corregidas](<capturas/reto-08/Captura de pantalla 2026-10-26 172814.png>)

## 6. Prueba de registro de cambios
La validación funcional consiste en generar un evento nuevo y localizarlo por su clave:

```bash
sudo touch /etc/cron.d/prueba
sudo rm /etc/cron.d/prueba
sleep 1
sudo ausearch -k cron -ts recent -i
```

El objetivo es identificar la ruta del archivo y la hora del cambio, no solo encontrar un registro antiguo. La siguiente imagen queda situada provisionalmente en este apartado; su contenido debe revisarse antes de atribuirle la prueba completa.

![Evidencia adicional pendiente de verificar en este paso](<capturas/reto-08/Captura de pantalla 2026-10-26 172056.png>)

## 7. Comprobación final y persistencia
Finalmente ejecuté la comprobación del script. La salida muestra las reglas activas y el resultado `SUPERADO`, una evidencia adicional de que el comprobador aceptó la configuración.

![Reglas cargadas y comprobación superada](<capturas/reto-08/Captura de pantalla 2026-10-26 172920.png>)

Para documentar también la persistencia, se debe mostrar el fichero guardado con:

```bash
sudo cat /etc/audit/rules.d/minpriv.rules
```

Las reglas activas no sustituyen la evidencia del fichero persistente, y el mensaje del comprobador no sustituye la prueba propia con un evento nuevo.

## Lección de seguridad
He aprendido que tener auditd activo no basta: hay que definir reglas para las zonas críticas, escribir correctamente sus claves y comprobar tanto los eventos nuevos como la configuración persistente.

## Alcance y evidencias
Actividad educativa en un entorno propio y autorizado. Las imágenes se conservan en su ubicación original; el nombre de la carpeta no divide la memoria en varias actividades. Las capturas provisionales se pueden mover o sustituir sin cambiar los pasos de la explicación.
