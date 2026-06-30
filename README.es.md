# Retired Hosts para BackupPC

🌐 **Idiomas:** [English](README.md) | **Español**

> Automatiza el ciclo de vida de los hosts retirados en BackupPC utilizando las políticas nativas de retención y mantiene el repositorio de backups libre de directorios huérfanos.

---

# Descripción

Administrar hosts retirados en BackupPC suele ser una tarea manual y repetitiva.

Si bien BackupPC dispone de un excelente mecanismo de retención de backups, no administra automáticamente el ciclo de vida completo de un host que ha sido retirado del servicio. A medida que una instalación de BackupPC crece, es habitual encontrar dos situaciones:

* Hosts que ya no reciben nuevos backups pero conservan históricos.
* Directorios de backups que permanecen en disco incluso después de eliminar un host de la configuración de BackupPC.

Ambas situaciones provocan un aumento gradual del espacio utilizado y generan tareas administrativas innecesarias.

**BackupPC Retired Hosts** automatiza este proceso aprovechando el funcionamiento nativo de BackupPC.

En lugar de modificar manualmente la política de retención de cada host o eliminar directorios de forma manual, el administrador únicamente debe deshabilitar los backups desde la interfaz web de BackupPC. El script detectará automáticamente esos hosts retirados y aplicará una política de retención reducida, permitiendo que BackupPC elimine los backups antiguos durante su proceso normal de housekeeping.

Además, el proyecto incorpora un modo de mantenimiento capaz de eliminar directorios huérfanos que BackupPC no elimina automáticamente cuando un host deja de existir en su configuración.

El resultado es un repositorio más limpio, menor consumo de almacenamiento y una administración mucho más simple.

---

# ¿Qué problema resuelve?

En instalaciones medianas y grandes de BackupPC es habitual que, con el paso del tiempo, se acumulen hosts retirados.

Sin una política de retiro definida, los administradores terminan realizando tareas como:

* Identificar qué hosts ya no se utilizan.
* Modificar manualmente las políticas de retención.
* Esperar la expiración de los backups.
* Eliminar directorios antiguos.
* Recuperar espacio en disco manualmente.
* Revisar periódicamente el repositorio en busca de residuales.

Todas estas tareas son repetitivas, consumen tiempo y, muchas veces, terminan olvidándose.

Este proyecto automatiza ese flujo operativo respetando completamente el funcionamiento nativo de BackupPC.

No reemplaza el mecanismo de retención de BackupPC, sino que lo complementa automatizando el retiro de hosts y eliminando residuales que BackupPC deja en el sistema de archivos una vez que un host ha sido eliminado de la configuración.

---

# Características

* Automatiza el retiro de hosts en BackupPC.
* Utiliza las políticas de retención nativas de BackupPC.
* Evita modificaciones manuales de retención.
* Reduce tareas administrativas repetitivas.
* Elimina directorios de backups huérfanos.
* Limpia residuales que BackupPC no elimina automáticamente.
* Ayuda a recuperar espacio en disco.
* Facilita la administración de instalaciones grandes.
* Puede ejecutarse automáticamente mediante cron.
* Utiliza un archivo de configuración externo.
* Diseñado para ejecutarse con el usuario **backuppc**.

---

# ¿Cómo funciona?

```text
                 El host deja de utilizarse
                           │
                           ▼
            BackupDisable = 1 o BackupDisable = 2
                           │
                           ▼
          retired_hosts --run (cron semanal)
                           │
                           ▼
      El script aplica la política de retiro
                           │
                           ▼
      BackupPC ejecuta su housekeeping normal
                           │
                           ▼
      Los backups expiran según la retención
                           │
                           ▼
      retired_hosts --cleanup-orphans
                           │
                           ▼
     Se eliminan directorios huérfanos restantes
```

El script **no elimina backups directamente**.

Su función consiste en preparar los hosts retirados para que BackupPC aplique su propia política de retención de forma totalmente automática.

De esta manera se mantiene el comportamiento esperado de BackupPC evitando tareas manuales.

---

# Requisitos

* BackupPC
* Bash
* Una instalación funcional de BackupPC
* Ejecución con el usuario **backuppc**
* Permisos para modificar los archivos de configuración de los hosts

---

# Instalación

Copiar el script dentro del directorio del usuario **backuppc**.

```bash
cp retired_hosts /home/backuppc/
chmod +x /home/backuppc/retired_hosts
```

El script está diseñado para ejecutarse con el usuario **backuppc**.

Se recomienda realizar una ejecución manual antes de programarlo mediante cron.

Ejemplo:

```bash
sudo -u backuppc /home/backuppc/retired_hosts --run
```

---

# Configuración

El script obtiene su configuración desde un archivo externo.

Ejemplo:

```bash
CONFDIR="/etc/BackupPC/pc"
HOSTSFILE="/etc/BackupPC/hosts"
PCDIR="/data0/backuppc/pc"
LOGDIR="/home/backuppc/log"
```

Mantener la configuración fuera del script permite reutilizar la misma versión en distintos servidores sin necesidad de modificar el código.

Las variables principales son:

| Variable    | Descripción                                                       |
| ----------- | ----------------------------------------------------------------- |
| `CONFDIR`   | Directorio donde BackupPC almacena la configuración de cada host. |
| `HOSTSFILE` | Archivo principal de hosts de BackupPC.                           |
| `PCDIR`     | Directorio donde se almacenan físicamente los backups.            |
| `LOGDIR`    | Directorio donde el script escribe los archivos de log.           |

Una vez creado el archivo de configuración, verificar que todas las rutas correspondan a la instalación de BackupPC antes de ejecutar el script.
---

# Referencia de comandos

## Aplicar la política de retención para hosts retirados

```bash
/home/backuppc/retired_hosts --run
```

Procesa todos los hosts marcados como retirados (`BackupDisable = 1` o `BackupDisable = 2`) y aplica automáticamente la política de retención reducida configurada para ellos.

El script **no elimina backups directamente**.

Su función consiste en preparar la configuración del host para que sea BackupPC quien elimine los backups expirados durante su proceso normal de housekeeping.

---

## Limpiar directorios huérfanos

```bash
/home/backuppc/retired_hosts --cleanup-orphans
```

Busca dentro del repositorio de BackupPC todos los directorios de backups que ya no pertenecen a ningún host configurado y los elimina de forma segura.

Esta operación es completamente independiente del proceso de retiro de hosts.

---

# Mantenimiento recomendado

Una planificación típica de mantenimiento podría ser:

| Tarea                             | Frecuencia                                          |
| --------------------------------- | --------------------------------------------------- |
| `retired_hosts --run`             | Semanal                                             |
| `retired_hosts --cleanup-orphans` | Mensual o después de eliminar hosts definitivamente |

Ejemplo:

### Todos los martes a las 12:02

```cron
2 12 * * 2 /home/backuppc/retired_hosts --run
```

### Primer domingo de cada mes a las 02:00

```cron
0 2 1-7 * 0 /home/backuppc/retired_hosts --cleanup-orphans
```

Con esta planificación, los hosts retirados son gestionados automáticamente cada semana y los residuales del repositorio se eliminan periódicamente sin intervención del administrador.

---

# Buenas prácticas

Para obtener el mejor resultado se recomienda:

* Retirar los hosts utilizando únicamente `BackupDisable = 1` o `BackupDisable = 2`.
* Evitar modificar manualmente las políticas de retención de hosts retirados.
* Permitir que BackupPC elimine los backups mediante su mecanismo nativo.
* Ejecutar `retired_hosts --run` semanalmente mediante cron.
* Ejecutar `--cleanup-orphans` como parte del mantenimiento periódico.
* Revisar los logs durante las primeras ejecuciones.
* Probar cualquier modificación antes de utilizarla en producción.

Siguiendo este procedimiento, el repositorio permanecerá limpio y consistente prácticamente sin intervención manual.

---

# Preguntas frecuentes

## ¿El script elimina backups?

No.

El script nunca elimina directamente los backups pertenecientes a un host retirado.

Únicamente modifica la configuración del host para que BackupPC aplique su política de retención y elimine automáticamente los backups expirados durante su housekeeping habitual.

---

## ¿Por qué no eliminar el host inmediatamente?

En muchos entornos es necesario conservar los backups durante algunos días o semanas después de retirar un servidor.

Este proyecto permite mantener ese histórico utilizando el mecanismo nativo de BackupPC sin necesidad de realizar tareas manuales.

---

## ¿Qué es un directorio huérfano?

Es un directorio que permanece dentro del repositorio de BackupPC aunque el host correspondiente ya no exista en la configuración del sistema.

Como BackupPC deja de administrar ese host, esos directorios pueden permanecer indefinidamente ocupando espacio en disco.

La opción `--cleanup-orphans` automatiza la eliminación de estos residuales.

---

## ¿Es seguro utilizar `--cleanup-orphans`?

Sí.

El script compara la configuración de BackupPC con el contenido del repositorio y únicamente elimina directorios que no pertenecen a ningún host configurado.

Los hosts válidos nunca son modificados.

---

## ¿Pueden programarse ambos modos?

Sí.

Son completamente independientes.

La recomendación general es:

* Ejecutar `--run` semanalmente.
* Ejecutar `--cleanup-orphans` una vez por mes.

---

# Beneficios operativos

Implementar **Retired Hosts para BackupPC** aporta ventajas importantes en instalaciones de cualquier tamaño:

* Reduce significativamente el tiempo dedicado al mantenimiento.
* Automatiza el retiro de hosts.
* Aprovecha completamente el mecanismo de retención nativo de BackupPC.
* Evita modificaciones manuales repetitivas.
* Elimina directorios residuales que BackupPC no limpia automáticamente.
* Ayuda a recuperar espacio en disco.
* Mantiene el repositorio limpio y organizado.
* Reduce la probabilidad de errores operativos.
* Permite una administración prácticamente desatendida mediante cron.

En lugar de reemplazar el funcionamiento de BackupPC, este proyecto lo complementa automatizando tareas que normalmente deben realizarse de forma manual.

---

# Autor

Developed by **mouleen**.

Infrastructure Engineer with extensive experience in Linux, BackupPC, virtualization, networking and automation.

This project was created to simplify BackupPC administration in large environments by automating repetitive maintenance tasks while preserving BackupPCs native behavior.

Contributions, suggestions and pull requests are welcome.

GitHub: https://github.com/mouleen


---

# Licencia

Este proyecto se distribuye bajo la licencia **MIT**.

Puede utilizarse, modificarse y redistribuirse libremente respetando los términos de dicha licencia.

Aunque el script fue diseñado para operar de forma segura junto con BackupPC, siempre se recomienda probarlo previamente en un entorno de pruebas antes de utilizarlo en producción.

---

# Agradecimientos

Este proyecto fue desarrollado con el objetivo de simplificar la administración diaria de BackupPC, automatizando tareas de mantenimiento que normalmente requieren intervención manual.

La intención no es reemplazar el funcionamiento de BackupPC, sino complementarlo, facilitando la gestión de hosts retirados y manteniendo el repositorio limpio, consistente y fácil de administrar incluso en instalaciones de gran tamaño.

