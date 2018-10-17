# Test du SparkFun RFID starter Kit
Le kit contient un lecteur RFID USB, un scanner RFID ID_12LA et deux tags préprogrammés format carte de crédit. Le tout fonctionne à 125kHz avec le protocole 64 bits EM4001/4100 qui contient entre autres 32 bits de données pour l'identification et 8 bits de checksum (chaine de dix caractères hexadécimaux).

[Lien SparkFun vers la description du kit](https://www.sparkfun.com/products/13198?_ga=2.32719358.1725444577.1539675252-164434784.1539675252)

[Lien SparkFun vers le tutoriel](https://learn.sparkfun.com/tutorials/sparkfun-rfid-starter-kit-hookup-guide?_ga=2.138162032.1725444577.1539675252-164434784.1539675252)

[Lien vers une description du protocole EM4100](http://www.priority1design.com.au/em4100_protocol.html)

Avec l'ID_12LA, la détection de la carte est faite à environ 4,5cm en suivant la normale à la surface de la puce. Avec un ID_20LA, acheté séparément, cette distance est portée à environ 7,5cm.

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

En utilisant le driver, les cartes du kit précédent sont détectés à environ 13cm en suivant la normale à la surface occupée par l'antenne. Ce kit supporte donc le protocole EM4100 mais également [l'ISO 11784](https://en.wikipedia.org/wiki/ISO_11784_%26_11785), le [HITAG S](https://www.united-access.com/sites/www.united-access.com/files/u2/HitagS_V11.pdf) ainsi qu'un protocole maison (le Phidget tag).

La tag doit être présent dans la zone de détection de l'antenne durant au moins 50ms pour être détecté.

Pour utiliser le lecteur, Phidget propose une API utilisable dans de nombreux langages (C, C#, Java, Python notamment) et très bien documentée.

### Utilisation avec Java
Une partie de la documentation peut être trouvée en suivant ce [lien](https://phidgets.com/docs/Language_-_Java). La librairie qui doit être liée à votre programme se trouve [ici](https://www.phidgets.com/downloads/phidget22/libraries/any/Phidget22Java.zip). Si vous voulez parcourir l'API, suivez simplement ce [lien](https://phidgets.com/?tier=3&catid=81&pcid=72&prodid=1023).

Voici le petit programme utilisé pour tester ce module :

```java
package fr.univamu.ism.rfid;

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
				System.out.println("From attachListener : " + attachEvent.getSource().getDeviceSerialNumber());
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
				System.out.println("From detachListener : " + detachEvent.getSource().getDeviceSerialNumber());
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
			System.out.println(dt);
			System.out.println("Tag out");
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
Les premières mesures montrent qu'en dessous d'une durée de présence du tag d'environ 215ms, la détection n'est pas faite.

[Python](https://phidgets.com/docs/Language_-_Python)
