http://wiki.ubuntuusers.de/SCREEN

Automatischer Aufruf
Um den Aufruf von "screen" beim Anmelden zu automatisieren (was bei einem Desktop-System evtl. nicht sinnvoll ist), kann die Datei .bash_profile im Homeverzeichnis wie folgt erweitert werden [3]:

 if  [ -z $STY ] && [ $TERM != "screen" ]; then
   /usr/bin/screen -xRR;
 else
   /usr/bin/screen -X hardstatus alwayslastline '[%H] %Lw%=%u %d.%m.%y %c '
 fi
ssh-Screen
Mit den folgenden Erweiterung am Ende der Datei .bash_profile wird automatisch mit einer screen-Sitzung verbunden, wenn die Verbindung per ssh erfolgt. Bestehende Verbindungen zu der Session werden ggf. getrennt:

if [ "$TERM" != "screen" ] && [ "$SSH_CONNECTION" != "" ]; then
   /usr/bin/screen -S sshscreen -d -R && exit
fi
Wird screen beendet oder getrennt, wird auch die ssh-Verbindung getrennt – so ist also nur eine ssh-Verbindung möglich. Das kann umgangen werden, indem man && exit weglässt.
