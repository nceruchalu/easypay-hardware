# MIFARE 
## MIFARE Classic Intro
http://www.ladyada.net/products/rfidnfc/mifare.html


## MIFARE DESFire
Model:                     MF3 1C D40
Free Space:                4K
Max. Apps:                 28
Max. Keys per app.:        14 (counting master key 0) [for apps AID != 0x00 ]
Max. Files per app.:       16
Crypto:                    DES, 3DES
Support for ISO 7816 Cmds: Very Limited

'Compatible Crypto Mode' must be used

MiFare DESFire are iso14443A compliant contactless smartcards, and support all
layers including iso4443-4. Think of these cards as memory cards
with access control. 

The new DESFire EV1 cards are supposed to address the flaws found in v0.6

### Introduction
* http://www.waazaa.org/download/fcd-14443-4.pdf
* http://ridrix.wordpress.com/2009/09/19/mifare-desfire-communication-example/

### MAD and MIFARE DESFIire
Every application has a 3 byte Application Identifier (AID) 

Available File Types:
Files within an application can be any of the following types
* Standard data files
* Backup data files
* Value files with backup
* Linear record files with backup
* Cyclic record files with backup

The AID 0xFFFFFF is reserved. It is used to store general issuer information.
* File 0x0 has to be a value file with free access for GetValue, holding the
  value 0x000003 indicating the MAD version 3.
* File 0x1 shall be configured as StandardDataFile with free read acess. This
  file holds the contact details of the Card Holder (user of the card) in CSV
  plain text. [Section 3.5 of MAD]
* File 0x2 shall be configured as StandardDataFile with free read access. This
  file holds the contact details of the Card publisher (owner of the PICC Master
  Key) in CSV plain text. [Section 3.8 of MAD]
 File 0x3 to 0xF are RFU (Reserved for Future Use) and shall not be used within
  DESFire AID 0xFFFFFF. PCDs shall ignore these files 0x3 to 0xF


### DESFire Command Protocols
* ISO 14443A-3
  - Request Type A (REQA)
  - Wake-Up (WUPA)
  - Anticollision & Select Cascade Level 1
  - Anticollision & Select Cascade Level 2

* ISO 14443-4:
  - Request for Answer To Select (RATS)
  - Protocol and Parameter Selection (PPS)
  - Frame Waiting Extension (WTX)

* T=CL ("CL" means "contact less"):
  - PICC Command Set


### DESFire PICC Commands
Depending on the version of the card, a DESFire card might support commands in
native, native-wrapped or iso78166-4 command set styles.
* Software version v0.4 does not support APDU (only native commands)
* v0.5 adds support for wrapping native commands inside ISO7816 style APDUs
* v0.6 adds ISO/IEC 7816 command set compatibility

Use native commands for full compatibility with all DESFire cards, and for
access to all commands.


## Native Command Mode
Most of these commands are one byte long, and the card responds with
"statusbyte + [optional data]". See Datasheet for more.

### Status byte examples
    00 : Command successful
    af : More data (send command 'af' to fetch remaining data)
    9d : Permission denied


### Communication flow
    --> To card
    <-- From card


## Example (using a blank DESFire v0.6 card):
### Get Version
    --> 60
    <-- af 04 01 01 00 02 18 05
    --> af
    <-- af 04 01 01 00 06 18 05
    --> af
    <-- 00 XX XX XX XX XX XX XX ZZ ZZ ZZ ZZ ZZ 05 06

The first response denotes the hardware related data: version is 0.2 (00 02),
and storage size is 0x18 (4096 bytes)
The second response denotes the software related data: version is 0.6 (00 06),
and storage size is 0x18 (4096 bytes)
The X's are the 7-byte UID
The Z's are the 5-byte batch number
05 = Calender week of prodution
06 = Production year


### Get Application Ids
    --> 6a
    <-- 00
    
No applications available (blank card)


### Select PICC Application
    --> 5a 00 00 00
    <-- 00
    
OK


### Get File IDs (for PICC Application)
    --> 6f
    <-- 9d
    
Permission denied


### Get Key Settings (For PICC Application)
    --> 45
    <-- 00 0f 01
    
0f = All bits in lower nibble are set, meaning configuration can be changed,
     CreateApplication/GetApplicationIDs/GetKeySettings can be performed without
     master key, and master key is changeable
01 = Only 1 key can exists for this application (the PICC application)


### Get Key version for key 00 (for PICC Application)
    --> 64 00
    <-- 00 00
    
The PICC master key version is 0x00


### Authentication with key 00 (for PICC Application)
    --> 0a 00
    <-- af a2 be cd 03 d8 46 cb 33
    --> af b0 cc bc ed 8f c8 38 c9 08 dc e2 4d 86 ca ec 3c
    <-- 00 76 73 d9 49 71 3f f2 d1

The example only showed authentication with the PICC application. In a real
world transaction you would typically select a specific AID (!= 00 00 00),
authenticate, and then read/write to files within that application.

After a successful authentication, further communication with the card is done
in plain/plain+MAC/encrypted+MAC, depending on the access bits for the
particular file.



## Authentication
Authentication is done using DES or Triple-DES depending on keysize. If key is
8 bytes: Single DES. If key is 16 bytes, and the first 8 bytes of the key are
different from the last 8 bytes: Triple-DES. The card terminal (PCD) always uses
DECRYPT_MODE (both when receiving and sending encrypted data), and the card
always uses ENCRYPT_MODE. However, the DESFire crypto is a bit different from
the normal DES/CBC scheme: The PCD uses DES "send mode" when sending data (xor
before DES), and the card uses DES "receive mode" when receiving data (xor after
DES). But when the PCD receives data, it uses normal DES/CBC mode (xor after
DES), and the card uses normal DES send mode when sending data (xor before DES).


## DESFire encryption
|                | Send encrypted data        | Receive encrypted data         |
|----------------|----------------------------|--------------------------------|
| PCD (DECRYPT)  | DES/CBC "send mode"        | Normal DES/CBC "receive mode"  |
| Card (ENCRYPT) | Normal DES/CBC "send mode" | DES/CBC "receive mode"         |


## TEST FUNCTION (USING AN SL032)
Test_SL032:
* initialize card type var. to an unknown value;
* send cmd(SelectCard[0x01])
* set timer to countdown 30ms
* wait in a loop until at least one of following happens:
  - UARTstatus is rxerr [determined by checksum analysis on rx'd data]
  - UARTstatus is rxsucc
  - timer times out
* exit the function if one of the following happens:
  - UARTstatus not rxsucc
  - Rx'd Command not SelectCard[0x01]
  - Rx'd Status is not 'Operation Success'[0x00]
* read the returned 'Data' byte and assign card type appropriately
  it is a DESFire if value is 6  

### The next steps are for a DESFire card 
* send cmd(RATSDesFire[0x20])
* set timer to countdown 300ms
* wait in a loop until one of the following happens:
  - UARTstatus is rxerr
  - UARTstatus is rxsucc
  - timer times out
* exit the function if one of the following happens:
  - UARTstatus is not rxsucc
  - Rx'd Command is not RATSDesFire[0x20]
  - Rx'd Status is not 'Operation Success'[0x00]
* send cmd(DesFireGetVersion);
* set timer to countdown 300ms
* wait in a loop until one of the following happens:
  - UARTstatus is rxerr
  - UARTstatus is rxsucc
  - timer times out
* exit the function if one of the following happens:
  - UARTstatus is not rxsucc
  - Rx'd Command is not 'T=CL exchange'[0x21]
  - Rx'd Status is not 'Operation Success'[0x00]
* note this, smart card status is actually the 5th received byte (index at 4)
  i.e. it is the first data byte in "Reader to Host" communication with SL032
* exit function if smart card status doesnt say 'Additional Frame'[0xAF]
* send cmd(GetAdditionalData)
* set timer to countdown 300ms
* wait in a loop until one of the following happens:
  - UARTstatus is rxerr
  - UARTstatus is rxsucc
  - timer times out
* exit the function if one of the following happens:
  - UARTstatus is not rxsucc
  - Rx'd Command is not 'T=CL exchange'[0x21]
  - Rx'd Status is not 'Operation Success'[0x00]
* exit function if smart card status doesnt say 'Additional Frame'[0xAF]
* send cmd(GetAdditionalData)
* set timer to countdown 300ms
* wait in a loop until one of the following happens:
  - UARTstatus is rxerr
  - UARTstatus is rxsucc
  - timer times out
* exit the function if one of the following happens:
  - UARTstatus is not rxsucc
  - Rx'd Command is not 'T=CL exchange'[0x21]
  - Rx'd Status is not 'Operation Success'[0x00]
* exit function if smart card status doesnt say 'Operation OK'[0x00]
* if you get this far in the function, turn on the buzzer... 
* turn on light for 30ms
* done!


## TODO
1. why do we have to getadditional data twice, after cmd(DesFireGetVersion)?
2. why do we set timer to 300ms?
