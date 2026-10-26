# Actividad de ciberseguridad: auditoría con auditd en Kali Linux

Práctica de ASIR sobre vigilancia de archivos críticos y aplicación del principio de mínimo privilegio en una máquina virtual propia.

## Objetivo y entorno
Configurar auditd en Kali Linux para registrar cambios relacionados con privilegios, verificar su funcionamiento y guardar reglas persistentes. Se utilizan permisos de administración y un snapshot previo.

## Situación inicial
Consultar `sudo auditctl -l` y probar un cambio en `/etc/cron.d/`. Sin la regla correspondiente, el cambio no genera un evento con la clave cron. Si se repite la práctica, pueden existir registros antiguos: revisar sus horas y no borrarlos.

## Configuración
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

`-w` selecciona la ruta, `-p wa` vigila escritura y cambios de atributos, y `-k` etiqueta los eventos. Así se controlan cambios en sudo, cuentas, tareas programadas y servicios.

Cargar y consultar las reglas:

```bash
sudo augenrules --load
sudo auditctl -l
```

## Validación y persistencia
Generar un evento nuevo y consultar el registro:

```bash
sudo touch /etc/cron.d/prueba
sudo rm /etc/cron.d/prueba
sleep 1
sudo ausearch -k cron -ts recent -i
```

Mostrar las reglas guardadas:

```bash
sudo cat /etc/audit/rules.d/minpriv.rules
```

Comprobar que la hora y la ruta del evento corresponden a la prueba y que las reglas están activas y guardadas. La descripción no sustituye a las salidas reales.

## Capturas de la actividad
Imágenes subidas durante la práctica, ordenadas por el nombre de archivo. Pueden incluir intentos y correcciones; no todas representan una validación final. El orden o su ubicación se puede ajustar después.

### Captura 1 · 16:18:49
![Captura de la práctica a las 16:18:49](<capturas/reto-08/Captura de pantalla 2026-10-26 161849.png>)

### Captura 2 · 16:30:28
![Captura de la práctica a las 16:30:28](<capturas/reto-08/Captura de pantalla 2026-10-26 163028.png>)

### Captura 3 · 16:33:57
![Captura de la práctica a las 16:33:57](<capturas/reto-08/Captura de pantalla 2026-10-26 163357.png>)

### Captura 4 · 17:18:48
![Captura de la práctica a las 17:18:48](<capturas/reto-08/Captura de pantalla 2026-10-26 171848.png>)

### Captura 5 · 17:19:39
![Captura de la práctica a las 17:19:39](<capturas/reto-08/Captura de pantalla 2026-10-26 171939.png>)

### Captura 6 · 17:20:56
![Captura de la práctica a las 17:20:56](<capturas/reto-08/Captura de pantalla 2026-10-26 172056.png>)

### Captura 7 · 17:25:42
![Captura de la práctica a las 17:25:42](<capturas/reto-08/Captura de pantalla 2026-10-26 172542.png>)

### Captura 8 · 17:28:14
![Captura de la práctica a las 17:28:14](<capturas/reto-08/Captura de pantalla 2026-10-26 172814.png>)

### Captura 9 · 17:29:20
![Captura de la práctica a las 17:29:20](<capturas/reto-08/Captura de pantalla 2026-10-26 172920.png>)

## Lección de seguridad
He aprendido que auditd necesita reglas específicas para vigilar las zonas críticas y que debo comprobar tanto los eventos nuevos como la persistencia de la configuración.

## Alcance
Actividad educativa exclusiva en un entorno propio o autorizado. No publicar contraseñas ni datos sensibles.
