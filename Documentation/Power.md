# Power


## Battery Terminology
Battery is Lithium Iron Phosphate (LiFePO4)

* **C-rate** is a measure of the rate at which a battery is discharged relative
  to  its maximum capacity. A 1C rate means that the discharge current will
  discharge the entire battery in 1 hour. For a batter with a capacity of
  100Amp-hrs this equates to a discharge current of 100 Amps. A 5C rate for this
  battery would be 500 Amps, and a 0.5C rate would be 50Amps.

* **Nominal Voltage**: The reported or reference voltage of the battery

* **Cut-off Voltage**: The minimum allowable voltage. This voltage generally
  defines the empty state of the battery

* **Capacity or Norminal Capacity (Ah for a specific C-rate)**: The coulometric
  capacity, the total Amp-hours available when the battery is discharge at a
  certain discharge current (specified as a C-rate) from 100 percent to cut-off
  Voltage.
  Capacity is calculated by multiplying the discharge current (in Amps) by the
  discharge time (in hours) and decreases with increasing C-rate.

* **Max Continuous Discharge Current**: The maximum current at which the battery
  can be discharged continuously.

* **Charge Voltage**: The voltage that the battery is charged to when charged to
  full capacity. Charging schemes generally consist of a Constant Current (CC)
  charging until the battery voltage reaches the charge voltage, then
  Constant Voltage (CV) charging, allowing the charge current to taper until it
  is very small.

* **Float Voltage**: the voltage at which the battery is maintained after being
  charged to 100% SOC (State of Charge) to maintain  that capacity by
  compensating for self-discharge of the battery.

* **(Recommended) Charge Current**: The ideal current at which the battery is
  initially charged (to roughly 70% SOC) under constant charging scheme before
  transitioning to constant voltage scheme.


## Battery Specs
* Supplier: BCC Electronics
* Capacity: 3Ah
* Nominal Voltage: 6.4V
* Max End-of-Charge Voltage: 7.2V
* Low Voltage Cut-off Voltage: 4V
* Standard Charge Current (during CC charging): 0.2C to 2C [0.6A to 6A]
* CV charging end current : < 0.1C
* Max Continuous Discharge current: 0.5C [1.5A]
* Max Discharge: 15C [45A]
* Cycle Life >= 2000 times
* Charge time with 0.2C charge current in CC mode: 5 hours
* Built in protection (over charge and over discharge) circuitry.
* Cell dimensions: 26 diameter x 65mm (each 26650 cell is just 3.2 V)
* Battery Pack dimensions: 28 x 55 x 69mm (2 cells side by side but in series)


## Battery Charger: (LT3652)
System still works in a no-battery condition, so for this put in a 100uF low ESR
non-ceramic (tantalum) capacitor from BAT pin to GND, in parallel with the 10uF
ceramic bypass cap. This 100uF cap also is needed when the battery is connected
to the charger with long wires.

* **LT3652_VIN** = VCHARGE = System input voltage
  - Solar Panel in production
  - Wall transformer in production
* **VCHARGE** has to be 3.3V greater than the programmed output battery float
  voltage for startup. But for operation, VCHARGE has to be 0.75V greater than
  VBATT
  - So on startup, VCHARGE >= 10.5V
  - For operation: VCHARGE >= 7.95

* **VIN_REG**: Used to program a minimum *operational* supply voltage, by using
  a resistive divider to set this pin at 2.7V
  
      Rin1/Rin2 = (VIN_Min/2.7) - 1
      Rin1/Rin2 = (8/2.7) - 1 = 1.96 ~ 2
      So using Rin1 and Rin2 as 200K and 100K

* **SHDN\\**: Charger is always on, so tied to VIN

* **CHRG\\**: sinks current (up to 10mA) when enabled so connect green LED here
  with cathode towards the pin, and anode at VIN

* **FAULT\\**: also sinks current so use red LED

* **TIMER**: disable by connecting to GND

* **VFB**: final voltage of this pin should be 3.3V. Using this, setup the final
  output voltage (float voltage) via a resistive divider.
  
      R1 = (VBATT * 2.5*10^5)/3.3 
      R2 = (R1 * 2.5*10^5)/(R1 - (2.5*10^5))  
      Use 7.1V for VBATT for battery safety to get R1=538K and R2 = 467K
      in current setup (470+536)/470 * 3.3V = 7.1V as final float voltage

* **NTC**: enable battery temp. monitor pin by connecting a 10KOhm, B=3380 NTC
  thermistor from this pin to GND.

* **BAT**: Connect to switch as a means of turning off the system

* **SENSE** Use to set the max. charge current (up to 2A)

        Rsense = 0.1/I_Chg(Max)
        Setting max charge current to 1A yields Rsense = 0.1 Ohms

* Inductor Selection
    L = (10*Rsense/Delta_I)*VBATT*[1- (VBATT/VIN_MAX)]
    Rsense = 0.1
    0.25 < Delta_I < 0.35
    VBATT = 7.1
    VIN_MAX = 18V
    ==> 12.3uH < L < 17.2 uH
    I picked middle ground of 15uH

* **SW**: The recitfier diode from this pin to GND is a Schotky diode with:
  - as low forward voltage as possible
  - rated to withstand reverse voltages greater than the max VIN
  - I_Diode(Max) = minimum average diode current rating
  
        I_Diode(Max) > I_Chg_Max*(VIN_Max - 0.7*VBATT)/VIN_Max
                     > 1*(18-0.7*7.1)/18
                     > 0.723A
                     for safety: > 1A
  - Picked a CDBA340L-G with reverse voltage rating of 40V, I_Diode(Max)=3A,
    Forward voltage = 0.4V at 3A
            

## VCC Generator (3.3V) [ADP3050ARZ-3.3]
Using "ADP3050 parts" for the design
* Vin (min) = 4.372V
* Vin (max) = 7.2V
* Vout = 3.3V
* Iout = 1A
* Design For = "Most Efficient"

Input voltage source is VBATT

However we are using an ADP3050-3.3 which is a fixed voltage buck converter,
so there will be some modifications based on the datasheet's Typical Application
Circuit (Figure 24)

* Inductor:

      L = [(VIN_Max - Vout)/Iripple] * (1/f_sw) * (Vout/VIN_Max)
      L = ((7.2 - 3.3)/0.3) * (1/200K) * (3.3/7.2)
      L = 30uH
  
      I_sw = Iout(max) + 0.5Iripple = 1 + 0.15 = 1.15A

  - So need an inductor with a saturation (or dc) current rating above
    1.2 * I_sw (1.38)

* Output Capacitor: 100uF low ESR

* Catch Diode:

      I_Diode_Avg = Iout * (Vin-Vout)/Vin
                  = 1 * (7.2-3.3)/7.2
                  = 0.54A

  - Current rating must be at least 0.54A
  - Low forward voltage
  - MBRS330T3G works

* Input Capacitor:

      Icin_Rms >= Iout * SQRT[(Vout/Vin) - (Vout/Vin)^2]
               >= 1 * SQRT[(3.3/7.2) - (3.3/7.2)^2]
               >= 0.5A
  - So RMS ripple current rating must be at least 0.5A
  - GRM31CR71E475K works (4.7uF)


## VCC38 Generator (3.8V) [TPS54331D]
Use "SwitcherPro"
* VinMin = 4.67V
* VinMax = 7.3V
* Vout   = 3.8V
* Iout   = 3A

Input Voltage Source is VBATT
