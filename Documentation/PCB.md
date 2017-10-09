# PCB
## Dimensions
Save 1mm on both sides to create margin for error/minimize shock on pcb

    Width :89mm [max is 91mm]
    Length:166mm [max is 168mm]

### Corner Cuts (Avoid Pillars)
Note 1mm taken off of values (this is explained in dimensions)

           +-----------------------------+
           |9.5mm                 9.5mm| 12 - 6 + 3.25 = 9.25 [buffer -> 10.5]
           |                             | 
    +-----+                             +-----+
    |  9mm                                9mm |
    |                                         |
    |                                         |
    
    /\/\/\/\/\/\/\/\/\/\/\/\/\/\/\/\/\/\/\/\/\/
    
    |                                         |
    |                                         |
    |  9mm                                9mm |
    +-----+                             +-----+
          |                             |  
          |23.5mm                 23.5mm| (91-80)/2 + 3.25 =8.75 [buffer ->10.0]
          |                             | 25 - 6 + 3.25 = 22.25 [buffer -> 24.5]
          +-----------------------------+



### Inner Cut (For Battery)
    L: 69mm [max]
    W: 55mm [max]
    H: 28mm [max]

                      55mm
                    ------->  
            +--------------+ /|\
            |              |  |69mm
            |              |  |
            |              |  
            |              |
            |              |
            |              |
            |              |
            |              |
     17mm   |              |
    ------->+--------------+
           /|\
    Bottom  |0mm
    Left    |

Bottom Left offset is calculated by:
    x = width(89 - 55)/2 = 17mm
    y = maximize pcb space =  0mm


## Board Layers
* Signal Layers: 2
* Power Planes: None = 0


## Via Style
* Thruhole vias only


## Component and Routing Tech
* Surface Mount components
* Components on both sides of the board: Yes


## Default Track and Via Sizes
Using 10/10 (track/clearance) rules for ease of hand soldering.
* Minimum Track Size: 10mil
* Minimum Via Width: 50mil
* Minimum Via HoleSize: 28mil
* Minimum Clearance:10mil


## Design -> Rules
* Mask -> SolderMask Expansion: 5mil
* Routing -> Width -> Power_Width:        
  - **Belongs to Net**: VBATT, VCC, VCC38, VCCLCD, VCCLCD_PIC, VCHARGE,
    VDDCORE, VREG_AUX, V_USIM, GND
    **Note**: Use an OR operand between the nets.
  - **Min/Preferred/Max Widths**: 25mil/25mil/100mil [Top & Bottom Layer]
* Minimum SolderMaskSliver: Default = 10mil    
* SMD Neck-Down: 100%                               

## Board Outline
Draw it with mechanical layer 2
