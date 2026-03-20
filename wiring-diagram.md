# Tennis Game Wiring Diagram - STM32F3Discovery

## Components

| Component | Quantity | Notes |
|---|---|---|
| STM32F3Discovery board | 1 | Onboard LED ring (PE8-PE15) used for ball |
| YSD-160AR4B-8 | 1 | Common anode, single digit, red 7-segment display |
| Push buttons | 2 | Player 1 and Player 2 rackets |
| LEDs | 2 | Winner indicators (e.g. green + blue) |
| 220 ohm resistors | 9 | 7 for display segments + 2 for winner LEDs |

## Pin Assignments

### Onboard (no wiring needed)

| STM32 Pin | Function |
|---|---|
| PE8 (LD4) | Ball LED 1 (NW) |
| PE9 (LD3) | Ball LED 2 (N) |
| PE10 (LD5) | Ball LED 3 (NE) |
| PE11 (LD7) | Ball LED 4 (E) |
| PE12 (LD9) | Ball LED 5 (SE) |
| PE13 (LD10) | Ball LED 6 (S) |
| PE14 (LD8) | Ball LED 7 (SW) |
| PE15 (LD6) | Ball LED 8 (W) |
| PA0 | Onboard user button (start/reset) |

### Push Buttons (active low, use internal pull-up)

| STM32 Pin | Function | Wiring |
|---|---|---|
| PA1 | Player 1 button | PA1 --> Button --> GND |
| PA2 | Player 2 button | PA2 --> Button --> GND |

### 7-Segment Display (YSD-160AR4B-8)

Common anode pins (3, 8) connect to **3.3V**.
Segments are active low: GPIO LOW = segment ON, GPIO HIGH = segment OFF.

| STM32 Pin | Resistor | Display Pin | Segment |
|---|---|---|---|
| PD0 | 220 ohm | Pin 7 | A (top) |
| PD1 | 220 ohm | Pin 6 | B (upper-right) |
| PD2 | 220 ohm | Pin 4 | C (lower-right) |
| PD3 | 220 ohm | Pin 2 | D (bottom) |
| PD4 | 220 ohm | Pin 1 | E (lower-left) |
| PD5 | 220 ohm | Pin 9 | F (upper-left) |
| PD6 | 220 ohm | Pin 10 | G (middle) |

### Winner Indicator LEDs (active high)

| STM32 Pin | Resistor | Function | Wiring |
|---|---|---|---|
| PC0 | 220 ohm | Player 1 winning | PC0 --> 220 ohm --> LED anode --> LED cathode --> GND |
| PC1 | 220 ohm | Player 2 winning | PC1 --> 220 ohm --> LED anode --> LED cathode --> GND |

## Display Pinout (YSD-160AR4B-8)

Viewed from the front, decimal point at bottom-right:

```
         10   9    8    7    6
         (G) (F)  (CA) (A)  (B)
        +------------------------+
        |         _a_            |
        |    f  |     |  b       |
        |       |_g___|          |
        |    e  |     |  c       |
        |       |_d___|   .dp    |
        +------------------------+
         (E) (D)  (CA) (C)  (DP)
          1    2    3    4    5
```

## Wiring Diagram

```
                        STM32F3Discovery
                    +---------------------+
                    |    [LED Ring]        |
                    |  PE15 ... PE8        |
                    |   (ball animation)   |
                    |                      |
     P1 Button      |                      |      P2 Button
     ___            |                      |            ___
    |   |---[PA1]---|          PA0         |---[PA2]---|   |
    |___|---[GND]---|    (onboard btn)     |---[GND]---|___|
                    |                      |
                    |                      |
                    |  PD0 PD1 PD2 PD3     |
                    |  PD4 PD5 PD6         |
                    |   |   |   |   |      |
                    |   |   |   |   |      |
                    |  PC0  PC1            |
                    |   |    |             |
                    +---|----|-------------+
                        |    |
            7-Seg       |    |        Winner LEDs
         +---------+    |    |
   3.3V--| 3,8(CA) |    |    |    PC0--[220]--[LED1]--GND
         |         |    |    |    PC1--[220]--[LED2]--GND
    PD0---[220]--7(A)   |    |
    PD1---[220]--6(B)   |    |
    PD2---[220]--4(C)   |    |
    PD3---[220]--2(D)   |    |
    PD4---[220]--1(E)   |    |
    PD5---[220]--9(F)   |    |
    PD6---[220]-10(G)   |    |
         +---------+
```

## GPIO Clocks Required

Enable in RCC_AHBENR (base: 0x40021000, offset: 0x14):

| GPIO Port | RCC_AHBENR Bit | Purpose |
|---|---|---|
| GPIOA | 17 | Push buttons |
| GPIOC | 19 | Winner LEDs |
| GPIOD | 20 | 7-segment display |
| GPIOE | 21 | Onboard LED ring |

## GPIO Configuration

| Port | Pins | MODER | PUPDR | Notes |
|---|---|---|---|---|
| GPIOA | PA1, PA2 | 00 (Input) | 01 (Pull-up) | Buttons, read 0 when pressed |
| GPIOC | PC0, PC1 | 01 (Output) | 00 (None) | Winner LEDs, HIGH = on |
| GPIOD | PD0-PD6 | 01 (Output) | 00 (None) | Display segments, LOW = on |
| GPIOE | PE8-PE15 | 01 (Output) | 00 (None) | Ball LEDs, HIGH = on |

## 7-Segment Digit Encoding (Common Anode)

Write these values to GPIOD_ODR bits [6:0] to display each digit.
(0 = segment ON, 1 = segment OFF)

| Digit | A(PD0) | B(PD1) | C(PD2) | D(PD3) | E(PD4) | F(PD5) | G(PD6) | Hex |
|---|---|---|---|---|---|---|---|---|
| 0 | 0 | 0 | 0 | 0 | 0 | 0 | 1 | 0x40 |
| 1 | 1 | 0 | 0 | 1 | 1 | 1 | 1 | 0x79 |
| 2 | 0 | 0 | 1 | 0 | 0 | 1 | 0 | 0x24 |
| 3 | 0 | 0 | 0 | 0 | 1 | 1 | 0 | 0x30 |
| 4 | 1 | 0 | 0 | 1 | 1 | 0 | 0 | 0x19 |
| 5 | 0 | 1 | 0 | 0 | 1 | 0 | 0 | 0x12 |
