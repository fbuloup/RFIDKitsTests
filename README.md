# Introduction

<p align="justify">
Cette page présente les tests effectués sur deux modules RFID. Le premier est le Sparkfun Stater Kit et le second est le Phidget RFID 1024_0. Les tests ont été effectués en utilsant les langages Java et Python. En conclusion est présenté une synthèse comparative de ces deux kits.
</p>

[Test du SparkFun RFID starter Kit](#testSparkfun)

* [Test du SparkFun RFID starter Kit](#testSparkfun)
	* [Avec Java](#testSparkfun_Java)
	* [Avec Python](#testSparkfun_Python)
* [Test du Phidget RFID 1024_0](#testPhidget) 
	* [Avec Java](#testPhidget_Java)
	* [Avec Python](#testPhidget_Python)
* [Conclusion](#conclusion) 

# Test du SparkFun RFID starter Kit <a id="testSparkfun"></a> 

<div align="justify">
Le kit contient un lecteur RFID USB, un scanner RFID ID_12LA et deux tags préprogrammés format carte de crédit. Le tout fonctionne à 125kHz avec le protocole 64 bits EM4001/4100 qui contient entre autres 32 bits de données pour l'identification et 8 bits de checksum (chaine de dix caractères hexadécimaux).
</div>

[Lien vers la description du kit](https://www.sparkfun.com/products/13198?_ga=2.32719358.1725444577.1539675252-164434784.1539675252)

[Lien SparkFun vers le tutoriel](https://learn.sparkfun.com/tutorials/sparkfun-rfid-starter-kit-hookup-guide?_ga=2.138162032.1725444577.1539675252-164434784.1539675252)

[Lien vers une description du protocole EM4100](http://www.priority1design.com.au/em4100_protocol.html)

<div align="justify">
Avec l'ID_12LA, la détection de la carte est faite à environ 4,5cm en suivant la normale à la surface de la puce. Avec un ID_20LA, acheté séparément, cette distance est portée à environ 7,5cm. Pour un test rapide sous Mac OS ou Linux, il suffit de lancer dans un terminal :
</div>

    screen /dev/tty.usbserial-A506LNUY
<div align="justify">
La connection série étant à 9600 bauds, inutile de le préciser sur la ligne de commande. En passant les cartes devant les lecteurs, on pourra voir l'identifiant s'afficher, comme ceci :
</div>

    5500378C20CE
    5500378223C3
    000000000101

<div align="justify">
Il est possible de supprimer le buzzer en enlevant la goute d'étain qui sert de jumper, sérigraphié Buzz sur le PCB ! En plus des bits relatifs au protocole EM4100, le module entoure la chaine de caractères des codes 0x02 (start of text) en début de chaine et 0x0A (CR), 0x0D (LF), 0x03 (end of text) en fin de chaine.
</div>

### Utilisation avec Java <a id="testSparkfun_Java"></a> 

Pour utiliser ce kit avec Java il faudra utiliser la bibliothèque native [JSSC](https://code.google.com/archive/p/java-simple-serial-connector/) pour pouvoir gérer le port série.

<details><summary>Cliquer pour voir le programme Java utilisé pour tester ce module</summary>

```java
package fr.univamu.ism.rfid;

import java.util.Scanner;

import jssc.SerialPort;
import jssc.SerialPortException;


public class SparkfunRFIDStarterKitTest extends Thread {
	
	private boolean terminate;
	private byte[] buffer = new byte[1024];
	private SerialPort serialPort;
	
	public SparkfunRFIDStarterKitTest() { 
	    try {
		terminate = false;
		serialPort = new SerialPort("/dev/tty.usbserial-A506LNUY");
	        serialPort.openPort();
	        serialPort.setParams(SerialPort.BAUDRATE_9600, SerialPort.DATABITS_8, SerialPort.STOPBITS_1, SerialPort.PARITY_NONE);
	    } catch (Exception e) {
		e.printStackTrace();
	    }
	}

	public void run() {
		while (!terminate && !isInterrupted()) {
			try {
				while ( ( buffer = serialPort.readBytes() ) != null ) {
				    System.out.print(new String(buffer,0,buffer.length));
				}
			} catch (Exception e) {
				terminate = true;
				e.printStackTrace();
			}
			
		}
		try {
			serialPort.closePort();
		} catch (SerialPortException e) {
			e.printStackTrace();
		}
	}
	
	public void terminate() {
		terminate = true;
	}

	public static void main(String[] args) {
		SparkfunRFIDStarterKitTest serialThread = new SparkfunRFIDStarterKitTest();
		serialThread.start();
		
		// Wait for user to press enter key to terminate program
		Scanner scanner = new Scanner(System.in);
		scanner.nextLine();
		scanner.close();
		
		serialThread.terminate();
	}
}
```
</details>
<br/>

### Utilisation avec Python <a id="testSparkfun_Python"></a> 

<details><summary>Cliquer pour voir le programme Python utilisé pour tester ce module</summary>

```python
# Create, configure and open serial port
import serial
ser = serial.Serial()
ser.baudrate = 9600 
ser.port = '/dev/tty.usbserial-A506LNUY'
ser.open()

# Detect 10 tag occurences and exit
n = 0
while True :
    line = ser.readline()
    print(line)
    n = n + 1
    if(n == 10):
        break
        
# Close serial port
ser.close()
```
</details>
<br/>
On obtient ce type de sortie console où l'on retrouve bien les caractères de début et de fin de texte ainsi que les caratères "retour chariot" et "saut de ligne" :

    b'\x025500378C20CE\r\n'
    b'\x03\x025500378223C3\r\n'
    b'\x03\x02000000000101\r\n'
    b'\x03\x02000000000303\r\n'
    b'\x03\x02000000000202\r\n'
    b'\x03\x025500378223C3\r\n'
    b'\x03\x025500378C20CE\r\n'
    b'\x03\x02000000000101\r\n'
    b'\x03\x02000000000303\r\n'
    b'\x03\x02000000000303\r\n'
    

# Test du Phidget RFID 1024_0 <a id="testPhidget"></a> 

Le kit contient un lecteur/enregistreur RFID USB. Il est donc possible de programmer soit même les identifiants des tags. Le tout fonctionne également à 125kHz mais avec différents protocoles, dont celui à 64 bits EM4001/4100 du kit précédent.

[Lien vers la description du kit](https://www.phidgets.com/?tier=3&catid=81&pcid=72&prodid=1023)

Un fois branché, ce kit n'apparaitra pas automatiquement dans les ports séries. Il faudra installer un driver pour l'OS préféré :

* [Mac OS](https://www.phidgets.com/docs/OS_-_macOS#Quick_Downloads)
* [Linux](https://www.phidgets.com/docs/OS_-_Linux#Quick_Downloads)
* [Windows](https://www.phidgets.com/docs/OS_-_Windows#Quick_Downloads)

En utilisant le driver, les cartes du kit précédent sont détectés à environ 13cm en suivant la normale à la surface occupée par l'antenne. Ce kit supporte donc le protocole EM4100 mais également [l'ISO 11784](https://en.wikipedia.org/wiki/ISO_11784_%26_11785), le [HITAG S](https://www.united-access.com/sites/www.united-access.com/files/u2/HitagS_V11.pdf) ainsi qu'un protocole maison (le Phidget tag).

Selon la documentation, Le tag doit être présent dans la zone de détection de l'antenne durant au moins 50ms pour être détecté (Cf. [Documentation - Onglet "User Guide"](https://www.phidgets.com/?tier=3&catid=81&pcid=72&prodid=1023) pour plus de précision). Voici l'extrait en question :

    Object Speed
    When trying to read tags, you should allow the tag to remain within detection range for at least 50ms. 
    Tags moving through the detection area faster than this may not register at all.

Pour utiliser le lecteur, Phidget propose une API utilisable dans de nombreux langages (C, C#, Java, Python notamment) et très bien documentée.

### Utilisation avec Java <a id="testPhidget_Java"></a> 

Une partie de la documentation peut être trouvée en suivant ce [lien](https://phidgets.com/docs/Language_-_Java). La librairie qui doit être liée au programme se trouve [ici](https://www.phidgets.com/downloads/phidget22/libraries/any/Phidget22Java.zip). Pour parcourir l'API, suivre ce [lien](https://phidgets.com/?tier=3&catid=81&pcid=72&prodid=1023) puis sélectionner l'onglet API.


<details><summary>Cliquer pour voir le programme Java utilisé pour tester ce module</summary>

```java
package fr.cnrs.ism.rfid;

import java.util.Scanner;

import com.phidget22.AttachEvent;
import com.phidget22.AttachListener;
import com.phidget22.DetachEvent;
import com.phidget22.DetachListener;
import com.phidget22.PhidgetException;
import com.phidget22.RFID;
import com.phidget22.RFIDTagEvent;
import com.phidget22.RFIDTagListener;
import com.phidget22.RFIDTagLostEvent;
import com.phidget22.RFIDTagLostListener;

public class Test {
	
	/*
	 * Some variables for tag detection duration measurement 
	 */
	private static long t, dt;
	
	/*
	 * Listener called at program start up if RFID Phidget is already connected
	 * or when Phidget is plugged in USB port
	 */
	private static AttachListener attachListener = new AttachListener() {
		@Override
		public void onAttach(AttachEvent attachEvent) {
			try {
				System.out.println("From attach listener : " + attachEvent.getSource().getDeviceSerialNumber());
			} catch (PhidgetException e) {
				e.printStackTrace();
			}
		}
	};
	
	/*
	 * Listener called when RFID Phidget is unplugged from USB port
	 */
	private static DetachListener detachListener = new DetachListener() {
		@Override
		public void onDetach(DetachEvent detachEvent) {
			try {
				System.out.println("From detach listener : " + detachEvent.getSource().getDeviceSerialNumber());
			} catch (PhidgetException e) {
				e.printStackTrace();
			}
		}
	};
	
	/*
	 * Listener called when a tag enters antenna's detection area
	 */
	private static RFIDTagListener rfidTagListener = new RFIDTagListener() {
		@Override
		public void onTag(RFIDTagEvent rfidTagEvent) {
			t = System.currentTimeMillis();
			System.out.println("Tag in");
		}
	};
	
	/*
	 * Listener called when a tag exits antenna's detection area
	 */
	private static RFIDTagLostListener rfidTagLostListener = new RFIDTagLostListener() {
		@Override
		public void onTagLost(RFIDTagLostEvent rfidTagLostEvent) {
			dt = System.currentTimeMillis() - t;
			System.out.println("Tag out : " + dt);
		}
	};

	/*
	 * Main method
	 */
	public static void main(String[] args) {
		try {
			// Create RFID object
			RFID rfid = new RFID();
			
			// Add listeners
			rfid.addAttachListener(attachListener);
			rfid.addDetachListener(detachListener);
			rfid.addTagListener(rfidTagListener);
			rfid.addTagLostListener(rfidTagLostListener);
			
			// Configure parameters : open channel number 0 of Phidget serial number 453467
			rfid.setDeviceSerialNumber(453467);
			rfid.setChannel(0);
			rfid.open(5000);
			
			// Wait for user to press enter key to terminate program
			Scanner scanner = new Scanner(System.in);
			scanner.nextLine();
			scanner.close();
			
			// Close RFID 
			rfid.close();
			
		} catch (PhidgetException e) {
			e.printStackTrace();
		}

	}

}
``` 
</details>
<br/>
Les premières mesures montrent que la détection n'est pas faite si la durée de présence du tag est plus petite que 215ms environ. Ce qui n'est pas en correspondance avec les spécifications !?

### Utilisation avec Python <a id="testPhidget_Python"></a> 

Suivre ce [lien](https://phidgets.com/docs/Language_-_Python) pour télécharger le module Python ou consulter la documentation. Voici le programme de test utilisé :

<details><summary>Cliquer pour voir le programme Python utilisé pour tester ce module</summary>
	
```python
# Used modules
from Phidget22.Devices.RFID import *
import datetime

# Variable for tag detection duration measurement 
t = datetime.datetime.now()

# Handler called at program start up if RFID Phidget is already connected
# or when Phidget is plugged in USB port
def onAttachHandler(self):
    print("From attach handler : " + str(self.getDeviceSerialNumber()))

# Handler called when RFID Phidget is unplugged from USB port
def onDetachHandler(self):
    print("From detach handler : " + str(self.getDeviceSerialNumber()))

# Handler called when a tag enters antenna's detection area
def onTagHandler(self, tag, protocol):
    global t
    print("Tag in")
    # Catch current time
    t = datetime.datetime.now() 

# Handler called when a tag exits antenna's detection area
def onTagLostHandler(self, tag, protocol):
    dt = datetime.datetime.now() - t
    print("Tag out : " + str(dt.microseconds/1000))

# Create RFID object
rfid =  RFID()

# Add handlers (listeners)
rfid.setOnAttachHandler(onAttachHandler)
rfid.setOnDetachHandler(onDetachHandler)
rfid.setOnTagHandler(onTagHandler)
rfid.setOnTagLostHandler(onTagLostHandler)

# Configure parameters : open channel number 0 of Phidget serial number 453467
rfid.setDeviceSerialNumber(453467)
rfid.setChannel(0)
rfid.open()

# Wait for user to press enter key to terminate program
input("Press Enter to terminate...")

# Close RFID 
rfid.close();
```
</details>
<br/>
Les mesures sont bien sûr équivalentes à celles effectuées avec le programme de test en Java.

# Conclusion <a id="conclusion"></a> 

En vrac avec avantages pour le kit RFID phidgets !

* les deux kit sont sensiblement au même prix
* le kit phidgets RFID possède des sorties numériques
* le kit phidgets RFID est plus sensible
* le kit phidgets RFID propose une API intéressante

À voir aussi (mais plus cher) :

https://learn.sparkfun.com/tutorials/simultaneous-rfid-tag-reader-hookup-guide


