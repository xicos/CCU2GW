# CCU2GW - use ccu2 as lan gateway 
## (only HM RF (like the [HM-LGW-O-TW-W-EU](https://www.elv.de/homematic-funk-lan-gateway.html)) - no HmIP!)

#### auf CCU2 per SSH anmelden und folgende Befehle ausführen:
```
# Dateisystem read/write mounten
mount -o remount,rw /

# Dateien herunterladen und Rechte anpassen
mkdir -p /usr/local/addons/
mv /firmware/fwmap /firmware/fwmap.orig
wget --no-check-certificate -q -O /firmware/fwmap https://raw.githubusercontent.com/jp112sdl/CCU2GW/master/fwmap
wget --no-check-certificate -q -O /usr/local/addons/hmlangw https://raw.githubusercontent.com/jp112sdl/CCU2GW/master/hmlangw
wget --no-check-certificate -q -O /etc/init.d/S61hmlangw https://raw.githubusercontent.com/jp112sdl/CCU2GW/master/S61hmlangw
chmod 755 /usr/local/addons/hmlangw
chmod 755 /etc/init.d/S61hmlangw

# Laufende Dienste stoppen
/etc/init.d/S70ReGaHss stop
/etc/init.d/S62HMServer stop
/etc/init.d/S61rfd stop
/etc/init.d/S60multimacd stop

# Funkmodul Firmware auf 1.4.1 updaten
eq3configcmd update-coprocessor -p /dev/mxs_auart_raw.0 -c -u -d /firmware

# Init-Skripte verschieben
export INITDIR=/etc/init.d/
export UNUSED_INITDIR=/etc/init.d_unused/
mkdir -p ${UNUSED_INITDIR}
mv ${INITDIR}S49hs485d ${UNUSED_INITDIR}
mv ${INITDIR}S50lighttpd ${UNUSED_INITDIR}
mv ${INITDIR}S50eq3configd ${UNUSED_INITDIR}
[[ -e ${INITDIR}S55cuxd ]] && mv ${INITDIR}S55cuxd ${UNUSED_INITDIR}
mv ${INITDIR}S58LGWFirmwareUpdate ${UNUSED_INITDIR}
mv ${INITDIR}S59SetLGWKey ${UNUSED_INITDIR}
mv ${INITDIR}S60hs485d ${UNUSED_INITDIR}
mv ${INITDIR}S60multimacd ${UNUSED_INITDIR}
mv ${INITDIR}S61rfd ${UNUSED_INITDIR}
mv ${INITDIR}S62HMServer ${UNUSED_INITDIR}
mv ${INITDIR}S70ReGaHss ${UNUSED_INITDIR}

mv /usr/local/etc/config/rc.d /usr/local/etc/config/rc.d_unused

mv /opt/mh/startup.sh /opt/mh/startup.sh_unused

# optional: 
# Seriennummer-Datei anlegen (nur notwendig wenn eine andere Seriennummer als CCU2GW0001 gewünscht wird)
# XXXXXXXXXX ändern in 10-stellige Seriennummer
echo XXXXXXXXXX > /usr/local/addons/serialnumber.txt

# Neustart durchführen
sync
reboot
```
- wenn das LAN-Gateway erfolgreich gestartet wurde, leuchtet die Info-LED dauerhaft
- die Standard-Seriennummer lautet CCU2GW0001
- _manueller Aufruf mit Debug-Option_ <br/>
`/usr/local/addons/hmlangw -D -n CCU2GW0001 -s /dev/mxs_auart_raw.0 -r -1`


<hr/>

Rückgängig machen (CCU2 wieder als CCU2 nutzen):

```
# Dateisystem read/write mounten
mount -o remount,rw /

# Dienst stoppen
/etc/init.d/S61hmlangw stop

# Init-Skripte verschieben und Dateien löschen
export INITDIR=/etc/init.d/
export UNUSED_INITDIR=/etc/init.d_unused/
mv ${UNUSED_INITDIR}S49hs485d ${INITDIR}
mv ${UNUSED_INITDIR}S50lighttpd ${INITDIR}
mv ${UNUSED_INITDIR}S50eq3configd ${INITDIR}
[[ -e ${UNUSED_INITDIR}S55cuxd ]] && mv ${UNUSED_INITDIR}S55cuxd ${INITDIR}
mv ${UNUSED_INITDIR}S58LGWFirmwareUpdate ${INITDIR}
mv ${UNUSED_INITDIR}S59SetLGWKey ${INITDIR}
mv ${UNUSED_INITDIR}S60hs485d ${INITDIR}
mv ${UNUSED_INITDIR}S60multimacd ${INITDIR}
mv ${UNUSED_INITDIR}S61rfd ${INITDIR}
mv ${UNUSED_INITDIR}S62HMServer ${INITDIR}
mv ${UNUSED_INITDIR}S70ReGaHss ${INITDIR}
rmdir ${UNUSED_INITDIR}
rm /etc/init.d/S61hmlangw
rm /usr/local/addons/hmlangw
rm /usr/local/addons/serialnumber.txt
mv /firmware/fwmap.orig /firmware/fwmap

mv /usr/local/etc/config/rc.d_unused /usr/local/etc/config/rc.d

mv /opt/mh/startup.sh_unused /opt/mh/startup.sh

# Neustart durchführen
sync
reboot
```

<hr/>


### selbst kompilieren:
#### Debian (9)
```
apt-get update
apt-get install libgcc-6-dev-armel-cross gcc-arm-linux-gnueabi g++-arm-linux-gnueabi libgcc1-armel-cross
git clone https://github.com/jp112sdl/CCU2GW/
cd CCU2GW/src
make
```
