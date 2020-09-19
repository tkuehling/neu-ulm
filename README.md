# RPI mit Raspbian installieren und anschließend vorbereiten:

# apt install wget build-essential subversion git python3 python3-pip libopenjp2-7

# Asterisk als Source-Package herunterladen

    cd /usr/local/src
    wget http://downloads.asterisk.org/pub/telephony/asterisk/asterisk-13-current.tar.gz
    tar xvzf asterisk-13-current.tar.gz
    cd asterisk-13.35.0

# Softmodem von https://github.com/proquar/asterisk-Softmodem als Asterisk-App herunterladen

# cd apps/
# wget https://raw.githubusercontent.com/proquar/asterisk-Softmodem/app_softmodem/app_softmodem.c
# cd ..

# Asterisk compilieren und konfigurieren

# make menuselect

Applications -> app_softmodem aktivieren

# make
# make install

## Basiskonfiguration anlegen

# make samples

## Asterisk konfigurieren

Tone Indications für Deutschland anpassen: asterisk/etc/indications.conf
SIP-User für die FRITZ!Box anlegen: asterisk/etc/sip.conf
Rufnummer 190 als BTX-Zentrale über Extensions konfigurieren: asterisk/etc/extensions.conf

Wichtig ist hierbei die Konfiguration zur Übergabe an die Softmodem-Applikation:

exten => 190,1,Answer()
        same => n,Wait(1)
        same => n,Softmodem(127.0.0.1, 8289, v(V23)ld(8)s(1)n)
        same => n,Hangup()

Damit die Übergabe an das Softmodem sich wie eine BTX-Zentrale verhält, wird V.23, 8 data bits und 1 stop bit verwendet. Nach dem Verbindungsaufbau wird mit der Option "n" ein NULL-Byte nach der Modem Carrier Detection (BTX spezifisch) gesendet.

# Neu-Ulm als BTX-Zentrale installieren

Zur Vorbereitung brauchen wir für python3 noch die passenden Erweiterungen:

# pip3 install beautifulsoup4
# pip3 install Pillow
# pip3 install feedparser

Repository clonen:

# cd /usr/local/src/
# git clone https://github.com/bildschirmtext/bildschirmtext.git

# Anbindung von neu-ulm über das Netzwerk mit Hilfe von socat

Damit neu-ulm über das Softmodem über Localhost erreicht werden kann, wird socat benutzt:

# cd /usr/local/src/bildschirmtext/server/
# socat TCP-LISTEN:8289,reuseaddr,fork 'exec:/usr/bin/python3 /usr/local/src/bildschirmtext/server/neu-ulm.py'

Sobald der Listener läuft, kann eine Einwahl getestet werden. Damit socat beim Boot gestartet wird, sind folgende Zeilen in der /etc/rc.local vor dem letzten exit hinzufügt worden:

# cd /usr/local/src/bildschirmtext/server/
# /usr/bin/socat TCP-LISTEN:8289,reuseaddr,fork 'exec:/usr/bin/python3 /usr/local/src/bildschirmtext/server/neu-ulm.py' &

# Troubleshooting

Falls es Probleme gibt, kann der Asterisk wie folgt in den DEBUG-Modus versetzt werden:

# rasterisk

*CLI> core set verbose 5
*CLI> core set debug 5
*CLI> module reload logger

Nach dem Troubleshooting sollten die Werte 5 noch wieder auf 0 gesetzt werden.