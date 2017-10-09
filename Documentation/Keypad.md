# Keypad

## Module
[27899PAR-ND](https://www.digikey.com/product-detail/en/parallax-inc/27899/27899PAR-ND/3523678)

## 4x4 Matrix Layout

                    |Col0       |Col1       |Col2       |Col3
                    |           |           |           |
                    |           |           |           |
             ___/ --x    ___/ --x    ___/ --x    ___/ --x
    Row0     | SW0  |    | SW1  |    | SW2  |    | SW3  |
    ---------x------|----x------|----x------|----x------|
                    |           |           |           |
                    |           |           |           |
             ___/ --x    ___/ --x    ___/ --x    ___/ --x
    Row1     | SW4  |    | SW5  |    | SW6  |    | SW7  |
    ---------x------|----x------|----x------|----x------|
                    |           |           |           |
                    |           |           |           |
             ___/ --x    ___/ --x    ___/ --x    ___/ --x
    Row2     | SW8  |    | SW9  |    | SW10 |    | SW11 |
    ---------x------|----x------|----x------|----x------|
                    |           |           |           |
                    |           |           |           |
             ___/ --x    ___/ --x    ___/ --x    ___/ --x
    Row3     | SW12 |    | SW13 |    | SW14 |    | SW15 |
    ---------x------|----x------|----x------|----x------|

Keyboards use a matrix with the rows and columns made up of wires. Each key acts
like a switch. When a key is pressed, its column and row wires make contact.

## Scanning to Detect Key Presses
I order to detect key presses, the MCU (Micro Controller Unit) will scan all
rows, activating each one by one. When a row is activated, the MCU detects
columns are activated.

The row values will be active low outputs from the MCU (Micro Controller Unit).
So they are inputs into the keypad.

The column values will be active low inputs into the MCU.
So they are outputs from the keypad.
Since these are active low they have to be puled up by default, to an inactive
state.

Assuming the data for the rows are sent out in nibbles having Row3 as MSB and
Row0 as LSB i.e. (Row3, Row2, Row1, Row0), when we scan we get:

    row 0, send out 1110
    row 1, send out 1101
    row 2, send out 1011
    row 3, send out 0111

If Switch8 happens to be pressed when row 2 is activated (1011), then the MCU
input from the columns nibble (Col3, Col2, Col1, Col0) will be 1110 because
Column 0 will now be active.

### Multiple Keys Pressed
#### In a situation where SW0,1,5 are pressed:
* When scanning Row0, 1110 is sent out and the input to the MCU is 1100 showing
  SW0 and SW1 are pressed.
* When scanning row 1, 1101 is sent out and the input to the MCU is still 1100
  implying SW4 and SW5 are pressed. But SW4 isn't pressed!

#### Here is Why (Intro to Ghosting)
* When Row1 is set to 0, SW5 being pressed pulls down Col1.
* However SW1 is connected to Col1, and SW1 connects Col1 to Row0. So Row0
  is activated to a 0, despite the MCU setting it to 1.
* Remember SW0 connects Row0 to Col0, so SW0 activates Col0 as it did when Row0
  was activated by the MCU.
* So the MCU reads in a keypad value with both Col1 and Col0 activated, hence
  1100. This is called **ghosting**!

#### Masking
We have defined ghosting. Masking is the reverse.
If SW0,1,4,5 were all pressed, and SW0 was released the MCU wouldn't know, as it
would again assume all 4 keys were still pressed. This is called **masking**!

#### Problem for MCU (And Solution)
Let's work with the ghosting case. The real issue was that Row0 was activated
by SW1 even when the MCU was attempting to deactivate it. This creates a
conflict and surely cannot be good for the MCU; it is indirectly setting an I/O
pin both active and inactive at the same time. You want to protect the MCU from
this, and you need diodes for this!

Position diodes on all the output pins of the MCU with the cathode (-ve) at the
MCU and the anode at the keypad.

This won't solve ghosting, but it protects the MCU from receving a 0 on Row0
when it is setting it to a 1. Remember how diodes work: the anode have to be at
a higher potential than the cathode by at least the forward voltage for any
current to flow.


                     |Vcc        |Vcc        |Vcc        |Vcc
                     |           |           |           |
                   |---|       |---|       |---|       |---|  
                   |   |       |   |       |   |       |   |
                   |10K|       |10K|       |10K|       |10K|
                   |   |       |   |       |   |       |   |
                   |---|       |---|       |---|       |---|
              Col0   |    Col1   |    Col2   |    Col3   |                
              -------|    -------|    -------|    -------|
                     |           |           |           |
              ___/ --x    ___/ --x    ___/ --x    ___/ --x
    Row0      | SW0  |    | SW1  |    | SW2  |    | SW3  |
    ----|<|---x------|----x------|----x------|----x------|
                     |           |           |           |
                     |           |           |           |
              ___/ --x    ___/ --x    ___/ --x    ___/ --x
    Row1      | SW4  |    | SW5  |    | SW6  |    | SW7  |
    ----|<|---x------|----x------|----x------|----x------|
                     |           |           |           |
                     |           |           |           |
              ___/ --x    ___/ --x    ___/ --x    ___/ --x
    Row2      | SW8  |    | SW9  |    | SW10 |    | SW11 |
    ----|<|---x------|----x------|----x------|----x------|
                     |           |           |           |
                     |           |           |           |
              ___/ --x    ___/ --x    ___/ --x    ___/ --x
    Row3      | SW12 |    | SW13 |    | SW14 |    | SW15 |
    ----|<|---x------|----x------|----x------|----x------|

Make your life easy and use 1N4148 switching diodes.

#### How to fix Aliasing (Ghosting and Masking)
The 4 diodes placed on the rows will protect the MCU but won't solve aliasing.
To solve ghosting let's look back at our example. We need that Row0 will not be
activated by SW1 while scanning Row1. To do this we need a diode between SW1 and
Row0, with the cathode at Row0.

This has to be built into they keypad. You cannot add this in external circuitry

##### Ref
http://www.dribin.org/dave/keyboard/one_html/

## Keypad Interrupts
To conserve power, the MCU should be able to go to sleep.
Before it does that, all rows should be activated by setting them to 0. This
means that when any key is pressed, the corresponding MCU input will be 0.
Remember that the MCU inputs are the keypad columns which are pulled-up to Vcc,
so a Diode-Resistor AND gate should be placed on the columns. The idea is once
one Column goes from 1 to 0, then the output of the gate will go from 1 to 0,
and create a pulse that should be connected to an MCU SYSTEM_WAKEUP pin.
            
                Vcc
                 |
               +---+
               |   |
               |10K|
               +---+
                 |
    Col0 --|<|---+------- SYSTEM WAKEUP (Col0 & Col1 & Col2 & Col 3)
                 |
    Col1 --|<|---+
                 |
    Col2 --|<|---+
                 |
    Col3 --|<|---+

##### Ref
https://en.wikipedia.org/wiki/Diode_logic#AND_logic_gate

## Connector
    Black           White

    R1 R2 R3 R4     C1 C2 C3 C4
