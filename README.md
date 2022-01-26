# First Step with MKR WAN 1310 and The Things Networks 
 Explaine how to begin with MKR WAN 1310 and The Things Networks

# Requirements

For beginning you need to create an account on TTN [here](https://account.thethingsnetwork.org/register).

And be covered by TTN gateway witch is probably the case if you are in a big city. If you are not covered, you need to create or buy a gateway.

# Prepare your project

First you need to install the necessary library and board manager.

Install board manager by searching mkr in board manager and install the first you see (Arduino SAMD Boards)

After you need the MKRWAN library witch manage the connection to the LoRa Chip. Search MKRWAN and install it. If you an early adopter you can use the V2 but it is in beta state when I wrote this and I'm not covering the use of V2 in this project.

# Update your device

Before doing anything, we need to be sure that the LoRa chip is up to date.

Connect your MKR WAN 1310 to your computer and load the example project "MKRWAN" > "MKRWANFWUpdate_standalone".

Upload it to your board and open the serial to see the update be applied. Once it's done we go further.

# Get the Device EUI

One of the most important information in LoRa protocol is the Device EUI called DevEUI. It is an identification number for your board. It is unique and should not be share. For retrieving it, load the example project "MKRWAN" > "FirstConfiguration", upload it to your board and open serial console.

You should see a line with "Your device EUI is:" and a series of number and letter.

Keep it somewhere because we need it for adding the device to TTN.

# Create TTN Application

Go to your TTN console and select the region that meet your location.

On the Applications tab, click on "+ Add application".

Entrer an Application ID for example "first-lorawan-application".

And Application name "First LoRaWAN Application"

# Add End device

End device is your Arduino. In your application, click "+ Add end device".

Select the Brand, Model like this:

Arduino SA, Arduino MKR WAN 1310

For the other information, use the latest version available and the profile that meet your region. You should see a picture of your Arduino and details about it.

Now we need to tell TTN what Frequency plan we want to use and what is our DevEUI. So for frequency plan, use the recommended or use by TTN frequency that meet your region.

For more details about frequency see [this frequency plans by country](https://www.thethingsnetwork.org/docs/lorawan/frequencies-by-country/).

Now paste your DevEUI in DevEUI input.

For AppEUI, click on "Fill with zeros" because we don't have AppEUI witch is an identifier for a custom registration server.

For AppKey, click on "Generate".

And for End device ID enter what you want example: "mkrwan1310-1".

Now click on "Register end device".

# First connection to LoRaWAN

Now that our device is registrer in the network, he is able to initiate a connection with TTN.

To do this, you have to edit one line of code.

It is the line N° 27. You need to put in "modem.begin()" the band of your region.

The band of your region can be: AS923, AU915, CN470, CN779, EU433, EU868, R920, IN865, US915, US915_HYBRID

After the line is good, you can upload the code to your board and open serial console.

It ask you if you are connecting via OTAA or ABP. Chose OTAA(1) by sending the number in serial.

Then send your AppEUI witch should be 0000000000000000.

And now, send the AppKey that you generate in the step before.

The board will try to connected to an TTN gateway and if it success then a payload that say "HeLoRA world!"

You should see "Message sent correctly!" in the serial console.

!!! It is not because you have "Error sending message :(" that the payload has not be send and receive by TTN. It's bug in TTN that is not sending in time the confirmation of message receive !!!

# See the result in TTN console

On TTN console, click on the Live data tab of your device, you should see something like this:

![TTN Console 1](https://pix.milkywan.fr/OqouxSHm.png)

PS: I'v got the translation from HEX, and I will explain how to get it in the next part of the project.

As you can see, the message has been sent and receive in HEX. That mean your device is registrer on TTN network and can send data.

The next step is to create your first project.

# The first project

On your IDE, create new project and name it like you want.

The project I create is simple:

- Open serial with computer
- Connect to LoRaWAN
- Wait that you send something to the serial
- Send data to LoRaWAN
- Print in serial the status of your message
- Loop with What that you send something to the serial

The first thing to put in your code is the library and the declaration of your LoRa chip and your appEui, appKey:
```c++
#include <MKRWAN.h>

LoRaModem modem;

String appEui = "0000000000000000";
String appKey = "********************************";
```

Next, create the setup void.

Open a serial for communication with your computer and wait for it to be UP.

After, open communication with LoRa chip.

Then, connect to LoRaWAN. I have create loop for keep retrying the connection until it works.

```c++
void setup() {
 Serial.begin(115200);
 while (!Serial);

 if (!modem.begin(EU868)) {
 Serial.println("Failed to start module");
 while (1);
  };

 int connected = modem.joinOTAA(appEui, appKey);
 while(!connected){
 Serial.println("Retry...");
 if(!modem.joinOTAA(appEui, appKey)){
 Serial.println("Fail");
    }
 else{
 break;
    }
  };
}
```

Now create a loop void.

First, wait for retrieve serial data from your computer.

Then send a payload to LoRaWAN.

And print to serial the state of the message.

```c++
void loop() {
 Serial.println();
 Serial.println("Send something in the serial console to send the payload");

 
while (!Serial.available());
  //This line is clearing the serial buffer
  String msg = Serial.readStringUntil('\n');

 Serial.println("Sending the payload 'Arduino' (HEX: 41726475696e6f)");

 Serial.println();
 int err;
 modem.setPort(1);
 modem.beginPacket();
 modem.print("Arduino");
 err = modem.endPacket(true);
 Serial.print("Message state: ");
 Serial.println(err);

 switch (err) {
     case -1:
         Serial.println("Timeout");
         break;

     case 1:
         Serial.println("Message sent !");
         break;
     default:
         Serial.print("Error while sending message with code: ");
         Serial.println(err);
         break;
}

 Serial.println();

} 
```

Now upload to your board and open serial.

Before going further, I tell you how to translate HEX to text in TTN console.

Go to TTN console and go to tab "Payload formatters" on your device.

In Uplink, Select Javascript in Formatter type and paste this:

```javascript
function Decoder(bytes, port) {
var result = ""; 
for (var i = 0; i < bytes.length; i++) { 
  result += String.fromCharCode(parseInt(bytes[i])); 
} 
return { payload: result, };
}
```
Save changes and return to your serial console.

Send something in serial console and wait for board to return status of your message.

Once it is sent, go to TTN console tab "Live data" of your device and you should see something like this:

![TTN Console 2](https://pix.milkywan.fr/2WOQSQvD.png)

# Code details

For every LoRaWAN project, you need this code:

Before anything:

- Include MKRWAN
- Declare LoRaModem
- Declare appui and appKey
```c++
#include <MKRWAN_v2.h>

LoRaModem modem;

String appEui = "0000000000000000";
String appKey = "********************************";
```
In setup:

- Open communication with LoRaModem
- Connect to LoRaWAN with JoinOTAA and keep retrying before going further

```c++
if (!modem.begin(EU868)) {
//Failed to start module
 while (1);
  };

 int connected = modem.joinOTAA(appEui, appKey);
 while(!connected){
//Retry
 if(!modem.joinOTAA(appEui, appKey)){
//Fail
    }
 else{
 break;
    }
  };
```
In your loop:

- SetPort to send data
- Open comunication
- Print your payload
- Close the communication

```c++
modem.setPort(1);
modem.beginPacket();
modem.print(yourPayload);
modem.endPacket();
```

The port is useful if your TTN application has a lot of end device and you need to separate different type of device for example port 1 for power consumption sensor, port 2 for temperature sensor, port 3 for water leak sensor...

# The end

Now you have all the needed information to create a project with an Arduino MKR WAN 1310 and TheThingsNetworks.

Have fun !