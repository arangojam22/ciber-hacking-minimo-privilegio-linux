# Ciberseguridad y hacking ético: mínimo privilegio en Linux

Actividad individual de ASIR · Equipo azul · Kali Linux · Retos 8–12.

## Objetivo y alcance
Detectar y corregir privilegios innecesarios, mantener el uso legítimo y justificar cada cambio con evidencias. Laboratorio exclusivo en una máquina virtual propia y autorizada.

## Entorno y preparación
- Kali Linux con permisos de administración e Internet.
- Snapshot antes de modificar la máquina.
- Ejecutar `sudo apt update`.
- Preparar cada reto por separado: `sudo bash /ruta/retos12.sh.txt preparar N`.
- Resolver en orden; el reto 12 requiere superar el 8.

Sustituir la ruta por la ubicación real del script. No ejecutar `limpiar` ni volver a preparar un reto mientras se recopilan sus evidencias.

## Estado de la documentación
El reto 8 se ha trabajado durante la práctica; falta adjuntar la evidencia final de la configuración corregida. Los retos 9–12 se presentan como guía pendiente de ejecución y validación. No se atribuyen resultados que todavía no se han demostrado.

## Reto 8 · Vigilancia con auditd
### Situación inicial
Inspeccionar `sudo auditctl -l`. Crear y eliminar `/etc/cron.d/prueba` y consultar `sudo ausearch -k cron -i`. Sin reglas no se registra ese cambio mediante la clave cron. Al repetir el laboratorio pueden aparecer eventos antiguos: revisar las horas y no borrar los registros.

### Solución
Guardar en `/etc/audit/rules.d/minpriv.rules`:
```text
-w /etc/sudoers -p wa -k sudoers
-w /etc/sudoers.d/ -p wa -k sudoers
-w /etc/passwd -p wa -k identidad
-w /etc/group -p wa -k identidad
-w /etc/shadow -p wa -k identidad
-w /etc/cron.d/ -p wa -k cron
-w /etc/systemd/system/ -p wa -k systemd
```
`-w` selecciona la ruta, `-p wa` vigila escritura y atributos, y `-k` etiqueta el evento. Cargar con `sudo augenrules --load`.

### Validación y evidencias
Mostrar el fichero persistente y `sudo auditctl -l`. Crear y borrar un fichero de prueba en cron; buscar el evento nuevo con `sudo ausearch -k cron -ts recent -i`. Añadir `comprobar 8` como prueba adicional.

Capturas: [carpeta del reto 8](capturas/reto-08/).

Lección: la auditoría necesita reglas específicas, persistentes y verificadas con eventos reales.

## Reto 9 · Auditoría de privilegios
### Situación inicial y corrección prevista
| Hallazgo | Inspección | Riesgo y corrección |
|---|---|---|
| SUID en `/var/lib/inventario/consulta` | `sudo find / -xdev -type f -perm -4000 2>/dev/null` y `dpkg -S /var/lib/inventario/consulta` | Privilegio innecesario: `sudo chmod u-s /var/lib/inventario/consulta`. |
| Cuenta `soporte_ext` con UID 0 | `awk -F: '$3==0' /etc/passwd` | Segundo root: `sudo userdel -f soporte_ext`, sin `-r`. |
| Sudo total para RR. HH. | `sudo grep -rvE '^\s*(#|$)' /etc/sudoers.d/` | Administración sin contraseña: retirar `/etc/sudoers.d/reto9_rrhh`. |
| Script de inicio modificable por todos | `sudo find /etc -xdev -type f -perm -o+w 2>/dev/null` | Inyección al iniciar sesión: propietario root y permisos 644 en `/etc/profile.d/empresa.sh`. |
| Nóminas legibles por todos | `sudo find /var/backups /srv -type f -perm -o+r 2>/dev/null` | Exposición de datos: `root:rrhh` y permisos 640 en `/var/backups/empresa/nominas_2025.csv`. |

### Validación y evidencias
Repetir las búsquedas y comprobar que desaparecen los cinco hallazgos, no todos los resultados legítimos del sistema. Confirmar que `ana_rrhh` puede leer la copia y que el script de inicio sigue existiendo. Añadir `comprobar 9`.

Capturas: [carpeta del reto 9](capturas/reto-09/).

Lección: corregir permisos no significa eliminar archivos necesarios ni retirar privilegios legítimos indiscriminadamente.

## Reto 10 · Carpeta compartida segura
### Situación inicial
Capturar `ls -ld /srv/proyecto`, `ls -l /srv/proyecto` y la lectura del informe mediante `sudo -u visitante cat /srv/proyecto/informe_ana.txt`.

### Solución propuesta
```bash
sudo chown root:proyecto /srv/proyecto
sudo chgrp proyecto /srv/proyecto/informe_ana.txt /srv/proyecto/notas_becario.txt
sudo chmod 660 /srv/proyecto/informe_ana.txt /srv/proyecto/notas_becario.txt
sudo chmod 3770 /srv/proyecto
sudo setfacl -d -m u::rwx,g::rwx,m::rwx,o::--- /srv/proyecto
```
Setgid hereda el grupo; sticky restringe el borrado de archivos ajenos; la ACL por defecto facilita la colaboración. Sticky no impide editar el contenido si se tiene permiso de escritura.

### Validación y evidencias
Confirmar que visitante no puede entrar. Crear un fichero temporal como Ana y probar que becario no puede borrarlo. Crear otro como becario, comprobar el grupo proyecto y editarlo como Ana. Mostrar `getfacl /srv/proyecto` y añadir `comprobar 10`.

Capturas: [carpeta del reto 10](capturas/reto-10/).

Lección: colaborar con seguridad requiere combinar grupo, permisos, setgid, sticky y ACL.

## Reto 11 · Endurecimiento de systemd
### Situación inicial
Capturar el usuario del proceso con `ps -o user,args -C python3` y la exposición con `systemd-analyze security informes`.

### Solución propuesta
Crear un drop-in mediante `sudo systemctl edit informes`, sin modificar la unidad original:
```ini
[Service]
DynamicUser=yes
NoNewPrivileges=yes
ProtectSystem=strict
ProtectHome=yes
PrivateTmp=yes
PrivateDevices=yes
CapabilityBoundingSet=
ProtectKernelTunables=yes
ProtectKernelModules=yes
ProtectControlGroups=yes
RestrictSUIDSGID=yes
RestrictRealtime=yes
LockPersonality=yes
```
Reiniciar con `sudo systemctl restart informes`. Medir después de cada cambio y añadir protecciones compatibles si la exposición sigue en 5.0 o más. No se garantiza una puntuación sin medirla en la VM.

### Validación y evidencias
Mostrar el usuario no root del proceso, exposición inferior a 5.0 y respuesta de `curl http://127.0.0.1:8081/`. Guardar el drop-in y añadir `comprobar 11`.

Capturas: [carpeta del reto 11](capturas/reto-11/).

Lección: aislar un servicio reduce el impacto de un fallo sin impedir su función legítima.

## Reto 12 · Investigación de un incidente
### Investigación previa a la limpieza
Ejecutar `sudo aureport -k -i --start today` y consultar `sudo ausearch -k CLAVE -i --start today` para identidad, sudoers y cron. Contrastar con `sudo journalctl --since today -t useradd -t usermod`.

### Línea temporal por completar con evidencia real
| Hora real | Evento | Evidencia |
|---|---|---|
| Pendiente | Creación de la cuenta | Auditoría y journal de useradd |
| Pendiente | Alta en sudo | Auditoría y journal de usermod |
| Pendiente | Creación de sudoers | Clave sudoers y ruta observada |
| Pendiente | Creación de cron | Clave cron y ruta observada |

Guardar `/root/reto12_informe.txt` con este formato, sustituyendo los marcadores por los datos investigados:
```text
CUENTA=cuenta_identificada
GRUPO=sudo
SUDOERS=/ruta/identificada
CRON=/ruta/identificada
```
Solo después de documentar la cronología, retirar la cuenta y los dos archivos identificados. No consultar los datos internos del comprobador para sustituir la investigación.

### Validación y evidencias
Probar que la cuenta y los archivos ya no existen, que `id kali` conserva sudo y que los registros de auditoría siguen disponibles. Añadir `comprobar 12`.

Capturas: [carpeta del reto 12](capturas/reto-12/).

Lección: preservar y analizar evidencias antes de limpiar permite reconstruir la persistencia del intruso.

## Cómo añadir capturas al README
Subir imágenes a la carpeta del reto correspondiente. Nombres sugeridos: `01-inicial.png`, `02-solucion.png`, `03-validacion.png` y `04-comprobacion.png`. Añadir las referencias cuando los archivos existan:
```markdown
![Estado inicial del reto 8](capturas/reto-08/01-inicial.png)
![Validación del reto 8](capturas/reto-08/03-validacion.png)
```
Cada captura debe mostrar prompt, comando y salida. Revisar que no contenga contraseñas, tokens ni datos personales reales.

## Entrega académica
GitHub organiza la práctica, pero no sustituye el PDF único solicitado. El PDF debe contener los cinco retos en orden, preparación, solución, validación propia y lección final. El mensaje SUPERADO es solo una prueba adicional.

## Material de referencia
Enunciado de la actividad y script `retos12.sh.txt` facilitado para el laboratorio. Las capturas y resultados reales se incorporarán durante la ejecución.
