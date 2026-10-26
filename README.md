# Actividad de ciberseguridad: auditoría con auditd en Kali Linux

Práctica de ASIR orientada a vigilar cambios en archivos críticos y aplicar el principio de mínimo privilegio en una máquina virtual de laboratorio.

## Objetivo
Configurar auditd para registrar modificaciones relacionadas con privilegios y comprobar que las reglas funcionan y permanecen guardadas.

## Entorno
- Kali Linux en una máquina virtual propia.
- Permisos de administración e Internet para instalar paquetes.
- Snapshot previo a los cambios.

## Situación inicial
Se consulta `sudo auditctl -l` y se realiza una prueba creando y eliminando un archivo en `/etc/cron.d/`. Sin reglas de vigilancia, ese cambio no genera un evento con la clave `cron`. Si se repite la práctica, pueden existir registros antiguos: hay que revisar sus horas sin borrarlos.

## Configuración
Guardar estas reglas en `/etc/audit/rules.d/minpriv.rules`:

```text
-w /etc/sudoers -p wa -k sudoers
-w /etc/sudoers.d/ -p wa -k sudoers
-w /etc/passwd -p wa -k identidad
-w /etc/group -p wa -k identidad
-w /etc/shadow -p wa -k identidad
-w /etc/cron.d/ -p wa -k cron
-w /etc/systemd/system/ -p wa -k systemd
```

`-w` indica la ruta vigilada, `-p wa` registra escritura y cambios de atributos, y `-k` asigna una etiqueta para buscar los eventos.

Cargar y consultar las reglas:

```bash
sudo augenrules --load
sudo auditctl -l
```

## Validación
Generar un cambio y consultar su registro:

```bash
sudo touch /etc/cron.d/prueba
sudo rm /etc/cron.d/prueba
sleep 1
sudo ausearch -k cron -ts recent -i
```

Comprobar también la configuración persistente:

```bash
sudo cat /etc/audit/rules.d/minpriv.rules
```

La evidencia debe mostrar un evento nuevo relacionado con el archivo de prueba y las reglas activas y guardadas. Los resultados finales se acreditan con las capturas, no solo con esta descripción.

## Capturas de la actividad
Guardar las imágenes directamente en [capturas/](capturas/). Nombres sugeridos: `01-inicial.png`, `02-reglas.png` y `03-validacion.png`.

Añadir las imágenes al README cuando estén subidas:

```markdown
![Situación inicial](capturas/01-inicial.png)
![Reglas de auditoría](capturas/02-reglas.png)
![Validación de los registros](capturas/03-validacion.png)
```

En cada captura deben verse el prompt, el comando y su salida. No incluir contraseñas ni información sensible.

## Lección de seguridad
He aprendido que auditd necesita reglas específicas para vigilar las zonas críticas y que debo verificar tanto el registro de eventos como la persistencia de la configuración.

## Alcance
Actividad educativa realizada exclusivamente en un entorno propio o autorizado.
