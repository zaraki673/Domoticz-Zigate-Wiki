# Legrand corner & faq

## Overview

Purpose is to present the finding around pairing Legrand/Netatmo devices (Celiane, Mosaic, dooxie to Zigate and Domoticz
A big thanks to @fgrimaldi who contributes heaviliy to this, a big thanks also to @Thorgal789

Quiet a lot of work have been done throught sniffing of the Zigbee traffic between the Legrand Hub and the devices as well as Try and Test approach.

## Preamble

By default some of the Legrand/Netamoi devices come with a basic firmware, which do not provide all features. 
In that context, I still feel that having the Legrand Hub is a good investement for upgrading your device, and then switching to Zigate to avoid Legrand cloud.

## WARNING

If you have the starter pack, do not reset it. It is factory preset with the corresponding remote and any reset will make it unusable and must be return to Legrand.

## Validated devices

* Smart plug - with instant power feedback (if upgraded via the Legrand Hub)
* Switch w/o neutral - with dimmer (if upgraded via Legrand Hub)
* Shutter switch with neutral - Open, Close and Stop
* Micromodule switch - On and Off
* Double gangs remote switch - On/Off/Dimmer
* Simple gang remote switch - On/Off/Dimmer
* Mobile outlet - On/Off/Power (instant power feedback if upgraded via the Legrand Hub)

## Not Validated
* The Home/Away remote which is provided with the Legrand Hub and which in addition of Home/Away scene allow to switch the Legrand Hub in pairing mode, do not show any Zigbee traffic .
* For the other Scenes related devices, they can be connected to the Zigate, but the only send Group+Scene commands in broadcast mode. 

## Prerequisites

* In order to enable the Dimmer feature on the Switch w/o neutral, you must have Zigate firmware 3.1b


## Add on

Some specific Legrand settings are accessible via the Web GUI settings page. 

* __EnableLedIfOn__ By enabling this setting, the device Led will be on ( blue ) when the device is On
* __EnableLedInDark__ By enabling this setting, the device Led will be on ( blue ) when the device is Off
* __EnableDimmer__ By enabling this setting, the Switch w/o neutral will get dimmer enable.

## Channels

* Channel 11 , pair all devices
* Channel 13, able to pair switch W/O neutral.
* Channel 15, able to pair micromodule and switch W/O Neutral

## Pairing process

1. Remove the device cover in order to get access to the Factory Reset hole (or button). Do not mix the factory reset hole/button with the led.

1. Enable the Zigate pairing mode and make sure that Domoticz accept new hardware is also on
1. Press the factory reset
   * after few seconds it should be flashing __green/red__ (stay pressing)
   * after around 8 seconds it should be flashing __red__ (the pairing process should be starting)
   * click immediatly on the factory reset once more
   * when turning the led is __green__ the pairing process is completed.
   
Alternative is :

When the led is red (not paired):
* press the reset button until it flash green 3 times
* when the led comes back to red, click once on the reset button
 
## Internals

### How the Pairing works

* At HUB reset (or OFF/ON), the HUB is sending a Write Attribute No Response and broadcast to Cluster 0x0000 , Attribute 0xf000 with a value of 0x00000000
* During the pairing of a device, this device, will request in unicast mode via a Read Attribute each of the existing main powered devices (including the controler ) for Cluster 0x0000 Attrribute 0xf000 value
* As described a little bit later 0x0000/0xf000 reflects the operation time since the latest reset or power on.

### Wireless devices

* Not able to use the Tap-Tap for pairing the wireless with an equipment. 
* Wireless devices are using a proprieatory Cluster 0xfc01 to send some commands. I suspect that Managemet of Pairing.
* Requires bindings of 0x0001, 0x000f and 0x0003 (in that order) - @Thorgal789 mentioned that 0x000f bind is not required.
* Use the Present Value ( cluster 0x000f / 0x0055 ) to get On/Off or Shutter Up and Down
* While with the Legrand HUB, there is no binding of cluster 0x0006 and 0x0008 for the remote switch, in order to get the Level Control when you do long press/long release, there is a need to bind Cluster 0x0006 and 0x0008

* The Remote coming with the hub is named : 'Master remote SW Home / Away', and was not able to pair with Zigate

#### Analysis of Double gangs remote switch

* Model ( 0x0000/0x0005) : Double gangs remote switch
* Manufacturer: ( 0x0000/0x0004 ): Legrand 1021
* Manufacturer Code: 
* ProfileID: 0104
* DeviceID 0104
* Application Version 0001
* Cluster IN Count: 02
* Cluster In 1: 0003 (Identify)
* Cluster In 2: 000f (Binary Input (Basic))
* Cluster OUT Count: 03
* Cluster Out 1: 0003 (Identify)
* Cluster Out 2: 0006 (On/Off)
* Cluster Out 3: 0008 (Level Control)


### Scenes

* From Master remote SW Home / Away
 * Away : GroupId 0xfff6, SceneId 0x01
 * Home : GroupId 0xfff7, SceneId 0x01
 * Wakeup: GroupdId: 0xfff5, SceneId 0x01
 * Going to Sleep: GroupId: 0xfff4, SceneId 0x01
 
### Groups when pairing a remote with a wired devices

* Looks that group numbering is started at 0xfeff and going down to 0xfe00  ( 255 Wireless Gang )

### Wired devices

* It looks like when using Groups for actions llike On/Off, there is not state return from the end device ! It might be related to the fact that Scene should be used instead of Group alone.

#### Plug-in Unit: Connected outlet

1. Add Group 0xfff7   
   1. Add SceneId 0x01 / Cluster 0x0006
1. Add Group 0xfff5
   1. Add SceneId 0x01 / Cluster 0x0006
1. Bind Cluster 0x0006
1. Configure Report 0x0006 / 0x0001
1. Bind Cluster 0x0b04
1. Configure Report 0x0b04 / 0x050b
1. Bind Cluster 0x0003


#### Micromodule: Micromodule switch

1. Add Group 0xfff6
   1. Add Scene 0x01 / Cluster 0x0006
1. Add Group 0xfff4
   1. Add Scene 0x01 / Cluster 0x0006
1. Bind 0x0006
1. Configure Report 0x0006 / 0x0001
1. Bind 0x000f
1. Bind 0x0003

#### Inter w/o neutral: Dimmer switch w/o neutral

1. Add Group 0xfff6
   1. Add Scene 0x01 / Cluster 0x0006
   1. Add Scene 0x01 / Cluster 0x0008
1. Add Group 0xfff4
   1. Add Scene 0x01 / Cluster 0x0006   
   1. Add Scene 0x01 / Cluster 0x0008
1. Bind 0x0006
1. Configure Report 0x0006 / 0x0001   (Bind and Configure Reporting on 0x0008, will be done when enabling dimmer)
1. Bind 0x000f
1. Bind 0x0003


#### Shutter: Shutter switch with neutral

1. Bind 0x000f
1. Bind 0x0003
1. Configure Report 0x0102

##### Commands

* Open: 0x05 value 0x64
* Close: 0x05 value 0x00
* (not open, not close): 0x05 value 0x02

Status return ( 0xff: Open, 0x00: Closed, 0x50: in between ) 
Ad Hoc status can be obtain by a read Attribute



### Other findings and decoding

#### Attributes

* Cluster 0x0000 / Attribute 0xf000 : Operating time in seconds

#### Suite  à Device Annoucement after Power Off/On

After a main Power Off, On, the device is sending a Device Annoucement , in response the Legrand HUB is sending a Wrte Attribute No Response in Broadcast
  * Command: 0x05 ( Write Attribute No Response )
  * Cluster: 0x0000
  * Attribute: 0xf000
  * Data Type: 0x23
  * Data: 0x00000000
  
#### Cluster 0xfc01
* With the help of @Thorgal789

| Device | Attribute | Values | Data Type | 
| ------ | --------- | ------ | --------- | 
| Dimmer switch w/o neutral | 0x0000 | 0x0101/0x0000 enable/disbale Dimmer mode ( cluster 0x0008 should be binded) | 0x09 (16bit data) |
| Dimmer switch w/o neutral | 0x0001 | 0x01/0x00 enable/disbale Led in Dark | 0x10 ( Bool )|
| Dimmer switch w/o neutral | 0x0002 | 0x01/0x00 enable/disable Led if On | 0x10 |
| Connected outlet          | 0x0002 | 0x01/0x00 enable/disable Led if On | 0x10 |
| Shutter switch with neutral | 0x0001 | 0x01/0x00 enable/disable Led if On | 0x10 |
| Fil Pilote                | 0x0000 | 0x0001/0x0002 enable/disable fil pilote | 0x0x09 (16 bit data)

__ATTENTION WIP / UNDER INVESTIGATIONS __

##### Comamnd: 0x02

* Device -> Hub
* Format: FCF ( 0x15 ), Manuf ( 0x1021), SQN, 0x02, Value: 1 byte
* Value:
  * 1byte: 0x00
  
##### Command: 0x03

* Device -> Hub
* Format: FCF ( 0x15 ), Manuf ( 0x1021), SQN, 0x03, Value: 1 byte
* Value:
  * 1byte: 0xff
  
  
##### Commande: 0x09 (Checked)

* Device -> Hub
* Format: FCF ( 0x15 ), Manuf ( 0x1021), SQN, 0x09, Value: 9 bytes
* Value:
  * 8 bytes are the MacAddress of the Source Device
  * 1 bytes seems to be a counter
  
* Expect : Command 0x0c from the Hub to Device


##### Command: 0x0a

* Device -> Hub
* Format: FCF ( 0x15 ),  Manuf ( 0x1021 ), SQN, 0x0a, Value: 12 bytes
* Value: eg. fefe a4091f0000740400 0101
  * 2 bytes: Most likely Group Membership
  * 8 bytes: IEEE ( Target address ??? ): 
  * 2 bytes: ( 0x0101 )
  
* Expect: Command 0x10 fromHub to Device

##### Commande: 0x0c

* Hub -> Device
* Format: FCF ( 0x1d ), Manuf ( 0x1021 ), SQN, 0x0c, Value: 3 bytes
* Value:
  * 2 bytes: Most Likely Group Membership ( 0xfefe, 0xfdfe, 0xf6fe, 0xf4fe, 0xfffe )
  * 1 byte: seems to correspond to counter from 0x09
  
##### Command: 0x0d

* Device -> Hub
* Format: FCF ( 0x1d ),  Manuf ( 0x1021 ), SQN, 0x0a, Value: 24 bytes
* Value: eg. ffffffffffffffffffffffffffffffffffffffffffffffff

  
* Expect: Command 0x10 fromHub to Device  
  
##### Command: 0x10

* Hub -> Device
* Format: FCF ( 0x1d ), Manuf ( 0x1021 ), SQN, 0x10, Value: 11 bytes
* Value: 00 0101 a4091f0000740400
  * 1 byte: status
  * 2 bytes: last 2 bytes from 0x0a command
  * 8 bytes: IEEE 00 0101 a4 09 1f 00 00 74 04 00
  
  

#### Cluster 0xfc40
* Contirbution from @Thorgal789

| Attribute | Heating Mode | Value | Data Type |
| --------- | ------------ | ----- | --------- |
| 0x0000    | Confort      | 0x00  | 0x30 (Enum8) |
|           | Contort -1   | 0x01  | 0x30 (Enum8) |
|           | Confort -2   | 0x02  | 0x30 (Enum8) |
|           | Eco          | 0x03  | 0x30 (Enum8) |
|           | Hors Gel     | 0x04  | 0x30 (Enum8) |
|           | Off          | 0x05  | 0x30 (Enum8) |  

#### Specific Commands

| Cluster | Command | Data Type | Data Value | Description |
| ------- | ------- | --------- | ---------- | ----------- |
| 0xfc01  | 0x02    | 1 byte    | 0x00       |             |


## OTA 

* Manfufacturer Code: 0x1021
* Block size for transfer: 64 bytes

| OTA Code | Manuf Code | Image Type | Product              | Size (B.) | App. Rel. | App Build | Stack Rel. | Stack Build |
| -------- | ---------- | ---------- | -------------------- | ----      | --------- | --------- | ---------- | ----------- |
|  NLF     | 0x1021     | 0x000e (14)| Switch w/o neural    | 229319    | 0         | 43        | 0          | 0           |
|  NLM     | 0x1021     | 0x0010 (16)| Micromodule          | 221655    | 0         | 32        | 0          | 0           |
|  NLP     | 0x1021     | 0x0011 (17)| Hub Plug             |           | 0         | 35        | 0          | 0           |
|  NLP     | 0x1021     | 0x0011 (17)| Mobile outlet        |           |           | 35        | 0          | 0           |
|  NLP     | 0x1021     | 0x0011 (17)| Connected outlet     |           |           | 35        | 0          | 0           |
|  NLT     | 0x1021     | 0x0012 (18)| Main Remote          |           | 0         | 33        | 0          | 0           |
|  NLV     | 0x1021     | 0x0013 (19)| Shutter              |           | 0         | 43        | 0          | 0           |
|  NLD     | 0x1021     | 0x0016 (22)| Double Remote Switch |           | 0         |  6        | 0          | 0           |
|          | 0x1021     |            | Remote Shutter       |           |           | 18        |            |             |
|  NLC     | 0x1021     | 0x0015 (21)| Cable outlet         |           |           | 10        |            |             |


* App. Rel. is shown on the MyHome application as Software Release
* Firmware are available : http://fw.netatmo.net/

More informations/scripts .... can be found here: https://github.com/pipiche38/Capture-OTA-from-Wireshark

## Pairing Mecansism

The HUB seems in addition of having Time Server, seems to store in the 0x0000/0xf000 the time since the HUB has been powered on. This attribute server as of time of operation.
At Pairing, the HUB is broadcasting this value to other devices, and the up coming devices is requesting this attribute also from the oher already connected devices. The Operating time ( 0x0000/0xf000) is reset to 0 after Power Off/On

 

## Other matters

* When Zigate is in pairing mode, then the Legrand device led is Green.
* When you just did the pairing, it is easy to add groupmembership to the device? However after a while I had issues to add new group membership. The workaround I found was. Power Off/On of the device, putting the Zigate in Pairing mode, and then I was able to add groupmembership.
## Reference

* https://faire-ca-soi-meme.fr/domotique/2018/09/24/test-legrand-celiane-by-netatmo-zigate/
* https://www.legrand.fr/catalogue/maison-connectee/prise-connectee
* https://www.legrand.fr/catalogue/maison-connectee/interrupteur-connecte
* https://github.com/Koenkk/zigbee2mqtt/issues/2399

