# Installation Etherpad-lite auf dem Pi

Grundlage dieser Anleitung ist https://github.com/ghoulmann/raspi-etherpad-lite

Zunächst die Paketedatenbank aktualisieren:


```bash
sudo apt-get -qq update
```

 

Dann ggf. fehlende Pakete nachinstallieren.


```bash
sudo apt-get install -y gzip git-core curl python libssl-dev pkg-config build-essential
```

Etherpad-lite benötigt node-js. Sollte installiert sein.


```bash
node --version
```

    v6.10.3


Variablen mit dem Ort und dem Port. Danach wird etherpad-lite geklonet.


```bash
target=/opt/etherpad-lite
port=8080
#git-clone to /opt/etherpad-lite
sudo git clone git://github.com/ether/etherpad-lite.git $target
```

    Klone nach '/opt/etherpad-lite'...
bjekte: 100% (29259/29259), 18.85 MiB | 1.77 MiB/s, Fertig.
    Löse Unterschiede auf: 100% (20877/20877), Fertig.
    Prüfe Konnektivität... Fertig.


Vorlage kopieren und danach den Port anpassen.


```bash
#Create settings.json
sudo cp $target/settings.json.template $target/settings.json
```


```bash
#configure etherpad
sudo sed -i "s|9001|$port|" $target/settings.json
```

Die Abhängigkeiten mit Skript nachinstallieren.


```bash
#Make sure dependencies are up to date
sudo $target/bin/installDeps.sh
```
Ein Benuzter der etherpad-lite als Dienst laufen lässt.

ToDo: Das Anlegen eines Homeverzeichnises sollte für einen Systembenutzer nicht notwendig sein. Evtl auf /opt/etherpad-lite ändern.


```bash
#useradd system user named etherpad-lite
sudo useradd -MrU etherpad-lite
sudo mkdir /home/etherpad-lite
sudo chowm etherpad-lite:
```


```bash
#create log directory
sudo mkdir -p /var/log/etherpad-lite/
```


```bash
#permissions
sudo chown -R etherpad-lite:etherpad-lite $target/
sudo chown -R etherpad-lite:etherpad-lite /var/log/etherpad-lite/
```

Erstellen eines Start-Stop (initV) Skriptes.


```bash
#configure daemon
sudo su -c 'cat > /etc/init.d/etherpad-lite <<"EOF"
#!/bin/sh
### BEGIN INIT INFO
# Provides:          etherpad-lite
# Required-Start:    $local_fs $remote_fs $network $syslog
# Required-Stop:     $local_fs $remote_fs $network $syslog
# Default-Start:     2 3 4 5
# Default-Stop:      0 1 6
# Short-Description: starts etherpad lite
# Description:       starts etherpad lite using start-stop-daemon
### END INIT INFO
PATH="/usr/local/sbin:/usr/local/bin:/sbin:/bin:/usr/sbin:/usr/bin:/opt/node/bin"
LOGFILE="/var/log/etherpad-lite/etherpad-lite.log"
EPLITE_DIR="#target#"
EPLITE_BIN="bin/safeRun.sh"
USER="etherpad-lite"
GROUP="etherpad-lite"
DESC="Etherpad Lite"
NAME="etherpad-lite"
set -e
. /lib/lsb/init-functions
start() {
  echo "Starting $DESC... "
  
	start-stop-daemon --start --chuid "$USER:$GROUP" --background --make-pidfile --pidfile /var/run/$NAME.pid --exec $EPLITE_DIR/$EPLITE_BIN -- $LOGFILE || true
  echo "done"
}
#We need this function to ensure the whole process tree will be killed
killtree() {
    local _pid=$1
    local _sig=${2-TERM}
    for _child in $(ps -o pid --no-headers --ppid ${_pid}); do
        killtree ${_child} ${_sig}
    done
    kill -${_sig} ${_pid}
}
stop() {
  echo "Stopping $DESC... "
   while test -d /proc/$(cat /var/run/$NAME.pid); do
    killtree $(cat /var/run/$NAME.pid) 15
    sleep 0.5
  done
  rm /var/run/$NAME.pid
  echo "done"
}
status() {
  status_of_proc -p /var/run/$NAME.pid "" "etherpad-lite" && exit 0 || exit $?
}
case "$1" in
  start)
	  start
	  ;;
  stop)
    stop
	  ;;
  restart)
	  stop
	  start
	  ;;
  status)
	  status
	  ;;
  *)
	  echo "Usage: $NAME {start|stop|restart|status}" >&2
	  exit 1
	  ;;
esac
exit 0
EOF'
```


```bash
#specify the etherpad install directory
sudo sed -i "s|#target#|$target|" /etc/init.d/etherpad-lite
```


```bash
#Make daemon file executeable
sudo chmod +x /etc/init.d/etherpad-lite
```


```bash
#Configure as a service
sudo update-rc.d etherpad-lite defaults
```


```bash
sudo /etc/init.d/etherpad-lite start
```

    Starting etherpad-lite (via systemctl): etherpad-lite.service.



```bash
sudo /etc/init.d/etherpad-lite status
```


```bash
nmap localhost
```

    
    Starting Nmap 6.47 ( http://nmap.org ) at 2017-06-08 14:01 CEST
    Nmap scan report for localhost (127.0.0.1)
    Host is up (0.0016s latency).
    Not shown: 994 closed ports
    PORT     STATE SERVICE
    22/tcp   open  ssh
    80/tcp   open  http
    3306/tcp open  mysql
    8080/tcp open  http-proxy
    8181/tcp open  unknown
    8888/tcp open  sun-answerbook
    
    Nmap done: 1 IP address (1 host up) scanned in 0.41 seconds


ToDo: Ändern auf mysql Datenbank.
    remote: Counting objects: 29259, donee
