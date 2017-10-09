# Data Module

## Overview
[SIM5218](http://simcom.ee/modules/wcdma-hspa/sim5218/), a 3G + GPS module.

### Bands
Quad-band GSM/GPRS/EDGE and UMTS that works on frequencies of:
GSM 850MHz, EGSM 900MHz, DCS 1800MHz, PCS 1900MHz and WCDMA 900MHz/2100MHz

These work for:
* US bands:
  - 2G: GSM 850, GSM 1900
  - 3G: UMTS 850, UMTS 1900, UMTS 1700, UMTS 2100

* Nigeria bands:
  - 2G: GSM 900, GSM 1800
  - 3G: UMTS 2100

### Antenna
The antenna connector (module side) is MURATA MM9329-2700

### Connector
70 pins board-to-board

### I/O
Inputs pulled down (unless explicitly instructed otherwise) to minimize noise.

### Sim Card
Keep USIM peripheral circuit close to USIM card socket.

### Power
Required input Voltage Source = 3.4V to 4.2V
Normal value is 3.8V (so we use that!)
        
Current drawn could get up to 2A, so power supply should be able to do that.
        
**Note** that supply power should NEVER drop below 3.4V even in a transmit burst
during which current consumption may rise up to 2A.

### Power On/Off
Will do this via the POWER_ON pin (PIC's DATAPOWER) because it seems you can't
turn on the SIM5218A with AT commands.
To do this I will use an NPN transistor switch (really an inverting buffer)

More on transistor switches here:
* http://www.technologystudent.com/elec1/transis1.htm
* http://hyperphysics.phy-astr.gsu.edu/hbase/electronic/transwitch.html
        
More on inverting buffers here:
* http://hyperphysics.phy-astr.gsu.edu/hbase/electronic/buffer.html#c4

**Note** that the POWER_ON pin has been pulled up in module, so the internal
pull up counts as the load on the Collector.

### Reset
Just like the Power_ON pin, it has been pulled up in module, so will use a
similar switch.

### UART
UART_RX line has VIHmax = VDD_EXT + 0.3 = 2.6V + 0.3 = 2.9V
This means we need to use a  buffer to convert the RX input from the PIC to an
appropriate voltage. This will be done with an SN74LVC1G126. The voltage level
of the buffer will be set by the module's VREG_AUX which is set at 2.85V.

The UART_TX line has VOHmax = VDD_EXT = 2.6V and VOHmin = VDD_EXT-0.2 = 2.4V
So this doesn't have to be buffered because it goes into the PIC and the PIC can
handle Voltages less than VCC (3.3V).
Also the PIC's VIHmin for Schmitt Trigger inputs is 0.8V, so 2.4V isnt a too low
VIN. The PIC's USART pins are Schmitt Trigger inputs.

### Camera
No need to pull low (to prevent floating inputs)
Will be configured as GPIO (AT+CCGSWT)
Inputs pulled low to prevent floating inputs -- waste of power

### GPIO4
Pull high to turn off flight mode (RF is working)
