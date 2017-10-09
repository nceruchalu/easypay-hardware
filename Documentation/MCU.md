# Micro Controller Unit (MCU)
## Module
[PIC18F67K22](http://www.microchip.com/wwwproducts/en/PIC18F67K22)

## Low Power Tips 'n Tricks
* Using I/O pins to power external components:
  - **LCD**: VCCLCD_PIC is a PIC I/O line powering LCD since LCD utilized less
    than 20mA
  - **RFID Module**: Work current could get up to 45mA, so cant power with a
    PIC I/O, so will send the Power Down commands and use the wakeup signal to
    restart it.
  - **Data Module**: Again draws high current (and uses 3.8V) so cant power via
    I/O, so will send Power Down and Power Up signals to it.

* Will be in sleep mode when not active, and will be woken up by SYSTEM_WAKEUP
  interrupt from the keypad

* Configuring Port Pins:
  Unused Port Pins: will be configured as output pins driving to either state
  (high or low).

* Using high-value pull up resistors (10K Ohms)


## External Clock (SG636PCE 18.4320MC0)
Using a high precision external clock of 18.432MHz to have 0% Baud Rate Error.
The desired Baud rate is 115.2KHz, to satisfy the default value of the data
module.


## On-Chip Regulator
This is enabled
