#  Control de Sensors i Càmara amb Rasberry Pi

El primer que hem de fer és instal·lar un sistema operatiu a la Raspberry Pi.

Aquest ordinador no porta disc dur, fa servir una petita tarjeta de memòria flash en format MicroSD (SDHC) igual que les que porten alguns mòbils. En aquesta targeta copiarem el sistema operatiu i ens farà la funció de disc dur.

Necessitem descarregar de Internet el sistema operatiu de la Raspberry. N'hi ha moltíssims, fins i tot es pot instal·lar una versió limitada de Windows anomenada Windows 10 IoT Core. Podem trobar totes les distribucions disponibles en aquest enllaç: [https://www.raspberrypi.org/downloads/]

Per aquest taller farem servir una distribució Linux anomenada Raspbian que podem trobar en aquest enllaç: [https://www.raspberrypi.org/downloads/raspbian/] 

Podem triar la versió "Raspbian Stretch with desktop" que instal·la només el sistema operatiu, després afegirem els programes necessaris. La versió anomenada "Raspbian Stretch with desktop and recommended software" té una mida de més de 4GB i porta molts programes que potser no farem servir.

El primer que cal fer és copiar la imatge descarregada a la tarjeta MicroSD, i això ho podem fer amb el programa Etcher, que té versions per Linux i per Windows. [https://www.balena.io/etcher/]

Un cop copiada a la tarjeta MicroSD la posem a la Raspberry i l'engeguem.

Un cop actualitzat el sistema, hem d'anar a habilitar la càmera. Això es fa amb el programa raspi-config
```
sudo raspi-config
```
Dins la opció "Interfacing Options", Aqui podem habilitar la càmara i l'accés per SSH i per VNC. El programa de VNC recomanat és RealVnc, que el podeu trobar a: https://www.realvnc.com/en/raspberrypi/  Si volem veure la imatge de la càmera per VNC cal anar a la configuració del servei (a la barra superior a la part dreta, al costat de Bluetooth i Xarxa) i al menú superior a la dreta triar "Options", "Troubleshooting", activar la opció "Enable direct capture mode" i aplicar els canvis.

També podem expandir la partició del sistema per tal que ocupi tota la mida de la MicroSD. Cal anar a "Advanced Options" i triar "A1 Expand File System"

A la configuració del sistema també es pot anar des del interficie gràfic "Preferències \ Raspberry Pi Configuration"


Un cop habilitada la càmara, ja la podem provar. La comanda per fer una foto és `raspistill` i per fer un vídeo `raspivid`. Si volem que la càmara ens mostri de forma contínua la imatge podem fer `raspistill -t 0`caldrà cancel·lar el procés per deixar de veure la imatge.



## Càmera de la Raspberry

Connectar la càmara a la RaspBerry amb el cable. La part blava va a l'interior i la part on es veuen els connectors ha d'estar a la banda del port HDMI. Primer cal axecar la peça blanca que fixarà el cable, posar-lo i despres fixar-lo baixant la peça blanca.

![Imgur](https://i.imgur.com/BeniiZW.jpg)


Habilitar la càmara des de la configuració de la raspberry.
```
sudo raspi-config
```

Habilitar la càmara i reiniciar la Raspberry.

sudo apt-get update
sudo apt-get upgrade

```
raspistill -o imatge.jpg
```

Si es vol fer un timelapse cal posar una entrada al crontab per tal que faci les fotos de forma regular. Es un bon consell posar un timestamp en el nom del fitxer.



## GPIO de la Raspberry

La Raspberry disposa de 40 pins per entrada i sortida, anomenats GPIO (general-purpose input/output) que tenen aquestes funcions:

![GPIO Raspberry Pi b+ 2 i 3](https://cdn-images-1.medium.com/max/1458/1*pcfeGQr_mUJrXDFDrdKMww.png  " GPIO Raspberry Pi b+ 2 i 3")



## Connectar DHT22

Per connectar aquest sensor cal una resistència d'entre 4.7K a 10k

![Connexió DHT22 ](https://tutorials-raspberrypi.de/wp-content/uploads/2015/08/luftfeuchtigkeit_DHT11_Steckplatine.png  "Connexió DHT22 ")


Per treballar aquest amb aquest sensor calen els paquets `build-essential python-dev python-openssl git` que en aquesta versió de raspbian ja venen instal·lats.

```
git clone https://github.com/adafruit/Adafruit_Python_DHT.git 
cd Adafruit_Python_DHT
sudo python setup.py install
cd examples
sudo ./AdafruitDHT.py 22 4
```

El programa AdafruitDHT requereix dos paràmetres, el primer és el tipus de sensor utilitzat (22 per DHT22) i el segon el pin GPIO on es troba connectat (Pin 7 GPIO 4). Si es volen enregistrar les dades, millor fer dos o tres mesures abans de prendre la mesura a enregistrar, ja que les primeres mesures mostren més variabilitat.


## Connectar un LED a la Raspberry.

Un LED és un dispositiu electrònic que només deixa passar la llum en un sentit, i que quan passa llum emeteix una senyal lluminosa. Té dos connexions, el positiu es diu ànode i el negatiu càtode.

![Anode_Catode](https://www.build-electronic-circuits.com/wp-content/uploads/2014/05/LED-anatomy-1024x455.png "Aànode i Càtode d'un LED")


Connectar el ànode del LED (pota llarga) a la sortida de 3.3V de la Raspberry (Pin 1) i afegir una resistència de  270R a 330R (amb més resistència farà poca llum)  i entre el càtode del LED (pota curta) i la terra (Pin 6). Si connectem el LED a l'inreves no farà llum.


![LED](https://projects.drogon.net/wp-content/uploads/2012/06/1led_bb1.jpg "LED")



D'aquesta manera el LED està sempre encés, si volem poder-lo apagar i encendre l'hem de connectar a un pin programable. 

Aquesta versió de Raspbian ja porta el programa raspi-gpio que ens permet gestionar els difetents pin del GPIO. 

Si fem servir el GPIO17 (Pin 11) hem de connectar el ànode a al pin 11 i llavors activant o desactivant aquest pin podrem controlar el LED. Per això cal definir aquest pin com "output" i després posar-lo a nivell high(1) o low(0)

```
raspi-gpio set 17 op
raspi-gpio set 17 dh
raspi-gpio set 17 dl
```

## Controlar una tira de LED amb un relé d'estat sólid.

Cal connectar el negatiu de la tira de led al negatiu del adaptador de 12V i el positiu del adaptador al negatiu del relé d'estat sòlid. El positiu del relé es connecta al positiu de la tira de led.

A la part del relé on posa 3V cal connectar la terra del GPIO i el GPIO que volguem utilitzar per controlar la tira de LED.

Al activar el GPIO s'encendrà la tira de LED
 


### Script per posar al crontab i fer fotos de forma periòdica


```
#!/bin/bash

DATA="$(date +'%Y%m%d-%H%M')"
TEMP="$(AdafruitDHT.py 22 22)"
TEMP="$(AdafruitDHT.py 22 22)"

echo "${DATA} ${TEMP}" >> /home/pi/registre.txt

raspi-gpio set 17 op
raspi-gpio set 17 dh
sleep 1

raspistill -o "/home/pi/TimeLapse/Foto-${DATA}.jpg"

raspi-gpio set 17 dl
```

### Gestionar la càmara des de Python

[https://projects.raspberrypi.org/en/projects/getting-started-with-picamera/5]

Script bàsic per mostrar la imatge de la càmara.

```
#!/usr/bin/env python
# -*- coding: utf-8 -*-
from picamera import PiCamera
from time import sleep

camera = PiCamera()

camera.start_preview()
sleep(2)
camera.stop_preview()
```

Un script python per capturar imatges i seqüències

```
#!/usr/bin/env python
# -*- coding: utf-8 -*-

from time import sleep
from picamera import PiCamera

# Defineix valors dels paràmetres de la càmera

camera = PiCamera()

camera.resolution = (1024, 768)
camera.iso = 800


camera.start_preview()
# Espera que s'estabilitzi la càmera
sleep(2)

# Fixa la resta de valors
camera.shutter_speed = camera.exposure_speed
camera.exposure_mode = 'off'

# Fixa el balanç de blancs
g = camera.awb_gains
camera.awb_mode = 'off'
camera.awb_gains = g

# Captura una imatge
camera.capture('unaImatge.jpg')

# Captura una seqüència
camera.capture_sequence(['img%02d.jpg' % i for i in range(10)])

# Captura un Timelapse
for filename in camera.capture_continuous('TimeLapse{counter:03d}.jpg'):
   sleep(1)


```




