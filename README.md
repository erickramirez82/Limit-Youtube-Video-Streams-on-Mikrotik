# Limit-Youtube-Video-Streams-on-Mikrotik
Limitar trafico de youtube

<img src="https://github.com/erickramirez82/Limit-Youtube-Video-Streams-on-Mikrotik/blob/master/mikrotik_vs_youtube.jpg?raw=true" />


1. Haga clic en IP> Firewall, seleccione la pestaña: Protocolos Layer7, y haga clic en el botón +, se mostrará como la imagen de abajo. y luego haz clic en Ok. Tendrás la nueva regla de los Protocolos Layer7 con el nombre de transmisión. Puede agregar cualquier otra transmisión de video url dentro de Regexp

<b> Opción 1 <b/>

```bash
/ip firewall layer7-protocol
add comment="" name=streaming regexp="^.*get.+\\.(c.youtube.com|cdn.dailymotion.com|metacafe.com|mccont.com).*\$"
```
<b> Opción 2 <b/>

```bash
/ip firewall layer7-protocol
add comment="" name=streaming regexp="videoplayback|video"
```

2. Aún en la ventana del firewall, seleccione la pestaña: Deshacer, aquí creará una nueva regla de mangle. Para más rápido, simplemente haga clic en el menú Nuevo terminal, copie el script mangle, haga clic con el botón derecho en la ventana del terminal y pegue allí.

<br>La secuencia de comandos mangle que debes insertar:

<b> Opción 1 <b/>

```bash
/ip firewall mangle
add action=mark-packet chain=prerouting \
comment="Mark Packet Streaming" disabled=no \
layer7-protocol=streaming new-packet-mark=streaming \
passthrough=no
```
<b> Opción 2 <b/>

```bash
/ip firewall mangle
add chain=prerouting action=mark-connection new-connection-mark=video_stream \
comment="Mark Streaming" passthrough=yes layer7-protocol=streaming in-interface=bridge-local
```

```bash
add chain=prerouting action=mark-packet new-packet-mark=video_stream_packet \
passthrough=yes connection-mark=video_stream
```

3. Queues. Lista de cola se mostrará. Seleccione la pestaña: Queues, aquí podrá crear una nueva regla de cola para la transmisión de video.

<b> Opción 1 <b/>

```bash
/queue tree add name="streaming" parent=global-out \
packet-mark=streaming limit-at=0 queue=default \
priority=8 max-limit=128k burst-limit=0 \
burst-threshold=0 burst-time=0s
```

<b> Opción 2 <b/>

```bash
/queue simple
add name=Limit_Video_Day target-addresses=192.168.88.0/24 \
direction=both disabled=no interface=bridge-local limit-at=128k/128k max-limit=256k/256k \
packet-marks=video_stream_packet parent=none priority=8 \
queue=default-small/default-small burst-limit=0/0 burst-threshold=0/0 burst-time=0s/0s \
total-queue=default-small
add name=Limit_Video_Night target-addresses=192.168.88.0/24 \
direction=both disabled=yes interface=bridge-local limit-at=0/3M max-limit=0/3M \
packet-marks=video_stream_packet parent=none priority=8 \
queue=default-small/default-small burst-limit=0/0 burst-threshold=0/0 burst-time=0s/0s \
total-queue=default-small
```


En este caso, queremos hacer diferencias de ancho de banda en Día y Noche

Day = 06:00am – 18:00pm – 256kbps. Max-Limit;<br>
Night = 18:00pm – 06:00am – 3Mbps. Max-Limit;

<b>Script</b><br>
System > Script

```bash
/system script
add name=Limit_Video_Day source="/queue simple enable Limit_Video_Day; \
/queue simple disable Limit_Video_Night"
add name=Limit_Video_Night source="/queue simple enable Limit_Video_Night; \
/queue simple disable Limit_Video_Day"
```

<b>Scheduler</b><br>
System > Scheduler

```bash
/system scheduler
add disabled=no interval=1d name=Limit_Video_Day on-event=Limit_Video_Day \
start-date=oct/10/2014 start-time=06:00:00
add disabled=no interval=1d name=Limit_Video_Night on-event=Limit_Video_Night \
start-date=oct/10/2014 start-time=18:00:00
```



