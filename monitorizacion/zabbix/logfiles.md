Mirar troubleshooting.md para ver como trabajar con el log de zabbix-server o zabbix-proxy.

# Loguear a journald

En vez de enviar a fichero, podemos enviar los logs de zabbix-server y proxy al journald.
De esta manera tendremos el nivel con el que se ha generado el log.
pri=2: crit
pri=3: err
pri=4: warning
pri=5: debug/trace
pri=6: info

También nos da las líneas correctamente, sin los cambios de línea que a veces se ven en el fichero de log y que complican el parseado.

# Logear debug a múltiples ficheros

<https://github.com/datadope-io/zabbix/tree/feature/log_multi_file>

# Configuración logging del server/proxy

Podemos sacar el logging a fichero, syslog o consola.

Lo seleccionaremos con:
LogType=file/system/console

Si usamos syslog (system), se enviará a la facility DAEMON con el programname = "zabbix_server"

/etc/rsyslog.d/zabbix.conf
:programname, isequal, "zabbix_server" -/var/log/zabbix/zabbix_server.log
:programname, isequal, "zabbix_server" stop

La segunda linea es para evitar que los logs de zabbix se envien a otra config tipo \*.info

systemctl restart rsyslog
systemctl restart zabbix-server

Si usamos systemd, los logs se están enviando primero al journal, siendo almacenados ahí y luego al rsyslog.
Esto no tiene pinta de ser muy buena idea y a parte satura mucho al journal (se le ve consumir una cpu bastante rato).
El journal tambien tira los mensajes al recibir demasiados:
journal: Suppressed 290762 messages from /system.slice/zabbix-server.service

Si escribimos a fichero, aún usando /dev/shm para evitar el disco, los procesos se ralentizan mucho por bloqueos a la hora de escribir al log.
Ejemplo de backtrace con el bloqueo:

# 0 0x00007fa9e3c7b6a7 in semop () from /lib64/libc.so.6

# 1 0x000000000048c6e3 in \*\*zbx_mutex_lock (filename=filename@entry=0x4fd47f "log.c", line=line@entry=238, mutex=mutex@entry=0x79790c <log_access>) at mutexs.c:216

# 2 0x00000000004564c0 in lock_log () at log.c:238

# 3 0x0000000000457045 in\*\*zbx_zabbix_log (level=level@entry=4, fmt=fmt@entry=0x4fe58e "In %s() data:EMPTY") at log.c:423

En el caso de usar syslog también se atascan. El bt

# 0 0x00007f06c2d23e6d in send () from /lib64/libc.so.6

# 1 0x00007f06c2d1cd31 in \*\*vsyslog_chk () from /lib64/libc.so.6

# 2 0x00007f06c2d1cff2 in\*\*syslog_chk () from /lib64/libc.so.6

# 3 0x0000000000457236 in syslog (**fmt=0x4ea938 "%s",**pri=7) at /usr/include/bits/syslog.h:31

# 4 \_\_zbx_zabbix_log (level=level@entry=4, fmt=fmt@entry=0x4fe58e "In %s() data:EMPTY") at log.c:548

Solución, los los de debug a un fichero distinto por cada pid.

Pruebas de performance.
Con 5 trappers, inyectando con 4 procesos en paralelo, 4000 métricas 10 veces cada uno, sin debug el tiempo total para cada uno era de unos 14".
Con debug al mismo fichero (lo estandar), 63"
Con debug a distintos ficheros, 17"

# Monitorizar logs

<https://www.zabbix.com/documentation/3.4/manual/config/items/itemtypes/log_items>
para journald mirar journald.md

Solo funciona con zabbix agent active

Podemos usar el agente de zabbix para monitorizar logs.

Usaremos una regexp para seleccionar lo que queremos buscar.

Una vez hecho el match, podemos elegir almacenar la línea matcheada o solo retorar un campo (para despues poner un trigger asociado a ese valor)

log[file,<regexp>,<encoding>,<maxlines>,<mode>,<output>]

log["/var/log/syslog","error"]
buscar la palabra "error" en ese fichero de log

Zabbix agent does not send more than maxlines of a log file per second. The limit prevents overloading of network and CPU resources and overrides the default value provided by MaxLinesPerSecond parameter in the agent configuration file.

Un ejemplo de trigger, there are several errors in the log for last 3 minutes:
{host:log["/var/log/syslog",ERROR].count(3m,ERROR,like)}>2

Extraer info del log:
log[file,<regexp>,<encoding>,<maxlines>,<mode>,<output>]

Ejemplo:
log[/var/log/syslog,"Total processors activated: ([0-9]+)",,,,\1]o

Fr Feb 07 2014 11:07:36.6690 */ Thread Id 1400 (GLEWF) large result buffer allocation - /Length: 437136/Entries: 5948/User: AUser/Form: CFG:ServiceLevelAgreement
log[/path/to/the/file,large result buffer allocation.*Entries: ([0-9]+),,,,\1]
extrae 5948
