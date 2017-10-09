# LCD
## Module
[NHD-0420AZ-FSW-GBW-33V3](http://www.newhavendisplay.com/specs/NHD-0420AZ-FSW-GBW-33V3.pdf)

## Backlight
Typical specs: 3.0V and 15mA

However system voltage is 3.3V, so the solution here is to burn off the extra
0.3V across a series resistor. If this resistor has 15mA passing through it,
to burn off 0.3V we need 0.3/0.015 Ohms = 20 Ohms.
Backlight series resistor will be 20 Ohms.


## Power
Supply Power Typical specs: 3.3V, 2mA
Supply Power Max specs:     5.5V, 3mA

This means the system uses a total of (15 + 3)mA between VDD and backlight.
Lets put this at 20mA tops.

The LCD power (VCCLCD) could come from:
* The system, VCC
* The MCU PIC, VCCLCD_PIC.
  The PIC MCU's PortA<7:6>, PortB, PortC can output up to 25mA.

Using VCCLCD_PIC has the advantage of being able to turn off the LCD module
via the PIC.
After some testing, will see if this is a viable solution.

## Contrast
Using a 10KOhms potentiometer to decide on the contrast.
Eventually this should be replaced by either a pull-up or pull-down, for 
maximum contrast.
