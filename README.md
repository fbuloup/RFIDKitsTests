# Test du SparkFun RFID starter Kit

Le kit contient un lecteur RFID USB, un scanner RFID ID_12LA et deux tags préprogrammés format carte de crédit. Le tout fonctionne à 125kHz avec le protocole 64 bits EM4001/4100 qui contient 32 bits de données pour l'identification, 8 bits de checksum.

[Lien SparkFun vers la description du kit](https://www.sparkfun.com/products/13198?_ga=2.32719358.1725444577.1539675252-164434784.1539675252)

[Lien SparkFun vers le tutoriel](https://learn.sparkfun.com/tutorials/sparkfun-rfid-starter-kit-hookup-guide?_ga=2.138162032.1725444577.1539675252-164434784.1539675252)

[Lien vers une description du protocole EM4100](http://www.priority1design.com.au/em4100_protocol.html)

Avec l'ID_12LA, la détection de la carte est faite à environ 4,5cm. Avec un ID_20LA, acheté séparément, cette distance est portée à environ 7,5cm.

Pour un test rapide sous Mac OS ou Linux, il suffit de lancer dans un terminal 

    screen /dev/tty.usbserial-A506LNUY
La connection série étant à 9600 bauds, inutile de le préciser sur la ligne de commande. En passant les cartes devant les lecteurs, on pourra voir l'identifiant s'afficher.

Il est possible de supprimer le buzzer en enlevant la goute d'étain qui sert de jumper, noté Buzz sur le PCB !

# Test du PhidgetRFID 1024_0
Le kit contient un lecteur/enregistreur RFID USB. Le tout fonctionne à 125kHz avec différents protocoles, dont celui à 64 bits EM4001/4100 du kit précédent.

[Lien vers la description du kit](https://www.phidgets.com/?tier=3&catid=81&pcid=72&prodid=1023)

Un fois branché, ce kit n'apparaitra pas automatiquement dans les ports séries. Il faudra installer un driver pour votre OS préféré :

* [Mac OS](https://www.phidgets.com/docs/OS_-_macOS#Quick_Downloads)
* [Linux](https://www.phidgets.com/docs/OS_-_Linux#Quick_Downloads)
* [Windows](https://www.phidgets.com/docs/OS_-_Windows#Quick_Downloads)
