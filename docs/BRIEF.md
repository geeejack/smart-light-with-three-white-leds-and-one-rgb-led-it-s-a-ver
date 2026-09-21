# Smart light with three white LEDs and one RGB LED It's a very warmth and colour. Photography quality.

Brief: three-board home lighting system

What it is

A 24 V LED light and a UK wall switch that controls it. There are three separate PCBs. They are not one board.

The control board sits with the lamp. A 24 V brick powers it. An ESP32-S3 on that board drives a lighting strip and can listen to a microphone. The light strip is a chain of dumb LED boards: no microcontroller, 24 V in, two clocked LED streams out. The touch panel replaces a UK single-gang switch. It powers itself from live and neutral in the back box, reads a finger on one slot, and sends the gesture to the control board over Wi-Fi. It never drives an LED and it never switches the lighting mains.

How the three boards meet

24 V brick ──► control board ──1×6 cable──► strip ──► next strip
                    ▲
                    │ Wi-Fi
                    │
mains L+N ──► touch panel

Lighting cable, 1×6, 2.54 mm. Pin 1 is the same end on every board so two strips butted end to end jumper straight across.







Pin



Name



What it is





1



+24V



From the brick, through the control board





2



GND



Power and signal return





3



TLC_CLK



TLC59711 clock, 5 V after the buffer





4



TLC_DATA



TLC59711 data, 5 V after the buffer





5



SK_CLK



SK9822 clock, 5 V after the buffer





6



SK_DATA



SK9822 data, 5 V after the buffer

The two protocols never share a wire. 5 V is made on each board that needs it and does not travel the cable. The touch panel is not on this connector.





Board 1 — control board

One per light. About 60 × 40 mm. 24 V DC only.

Must do





Take 24 V from a barrel jack, centre positive. Jack is DC-005-5A-2.0, LCSC C381116. Pin 1 is centre, +24 V. Pin 2 is the sleeve, GND. Pin 3 stays open.



Protect the input: fuse in series with 24 V, a series Schottky for reverse polarity, an SMAJ28A from 24 V to GND.



Make 5 V with an AP63205WU-7 (C2071056) and 3.3 V with an AP63203WU-7 (C780769). 6.8 µH, 3 A inductors (C57254). ESP32 Wi-Fi transmit peaks near 355 mA at 3.3 V, so the 3.3 V rail must be rated at least 500 mA.



Run an ESP32-S3-WROOM-1U-N8, LCSC C2980297. U.FL antenna, not a PCB antenna. 8 MB flash, no PSRAM.



Program it from USB-C, TYPE-C-31-M-12 (C165948). D− is GPIO19, D+ is GPIO20, each through 22 Ω. Each CC pin has 5.1 kΩ to GND. VBUS is not connected to the 5 V rail. The board is powered from the barrel jack while flashing.



EN and BOOT buttons to GND, each with 10 kΩ to 3.3 V. EN has 1 µF to GND.



Shift four GPIO lines from 3.3 V to 5 V with one 74HCT125D (C5962), powered from the 5 V rail, outputs enabled. SK9822 clock and data, and TLC59711 clock and data. 33 Ω in series on each line after the buffer, then onto the 1×6. SK clock and data may be GPIO14 and GPIO21. Any free GPIO pair is acceptable. Do not use GPIO0, GPIO3, or GPIO19/20.



Bring the 1×6 out as the lighting connector, pinout in the table above. Header is C37208 if fitted. The six holes are the interface.



Bring a second 1×6 out for a microphone breakout, unpopulated until a mic is chosen: 1 = 3.3 V, 2 = GND, 3 = GPIO4, 4 = GPIO5, 5 = GPIO6, 6 = 5 V.



Budgets





Input voltage: 24 V nominal, survive a 28 V TVS clamp.



3.3 V rail current: at least 500 mA continuous.



5 V rail: the 74HCT125, the mic header, and nothing else heavy.



Board size: 60 mm × 40 mm.



Lighting connector current: a 2.54 mm pin is about 3 A. A long strip run is fed by injecting 24 V along the chain, not by one header pin.



Constraints





2-layer, 1.6 mm, 1 oz, lead-free HASL, tented vias. JLCPCB / LCSC parts.



Parts named above are already chosen. Keep the LCSC numbers.



USB-C is data only.



No antenna keep-out on the module itself. The antenna is a U.FL pigtail. Keep metal off the whip.



Out of scope





Mains, a Hi-Link, a relay, or any connection to the lighting circuit’s live.



Touch electrodes, and any 10 kΩ series resistors meant for touch.



A second microcontroller.



Tying barrel pin 3 to GND.



Back-feeding the 5 V buck onto USB VBUS.



Driving the LEDs directly. This board only sends 24 V and the two clocked streams.





Board 2 — light strip

A chain of LED boards. No microcontroller. Each board takes the 1×6 in and passes it out.

Must do





Accept the 1×6 pinout above on both ends. Pin 1 is on the same long edge at both ends.



Pour 24 V and GND the length of the board. 5 V is made on the board from 24 V and never leaves it.



Drive the white LEDs with a TLC59711. Clock and data enter on pins 3 and 4, 33 Ω at the input, and leave from the last chip’s clock and data outputs onto the OUT header.



Drive per-pixel RGB with SK9822-EC20 (C2909059) on 5 V. Clock and data enter on pins 5 and 6, 33 Ω at the input, and leave from the last pixel.



Keep the two daisies separate. A TLC59711 frame is not an SK9822 frame.



Power every LED from the local 5 V rail. White LED anodes and SK9822 VDD are 5 V. No LED pad sees 24 V.



Allow the plastic header to be left off, so boards can be butted and the six holes jumpered.



Budgets





One board is a few watts at 24 V, not tens of watts.



Inject 24 V and GND every few boards on a long run. Do not pull a whole stair through one 2.54 mm pin.



SK9822 and TLC59711 logic is 5 V. The control board already level-shifts. This board does not level-shift.



Constraints





2-layer. Narrow enough to sit in a stair lip or a lamp extrusion. 1×6 pitch is 2.54 mm, so the short edge has to fit six holes.



LCSC parts. TLC59711 for white, SK9822-EC20 for RGB.



A local buck from 24 V to 5 V, fused on its own so a shorted buck does not crowbar the 24 V pour.



Out of scope





An ESP32 or any other MCU.



Mains, a relay, or the touch panel.



Sharing one clock or one data line between TLC59711 and SK9822.



24 V on an LED anode.



Mixing this pinout with a round module. Rounds are a different connector and are not part of this build.





Board 3 — touch panel

A UK single-gang wall plate, BS 4662, 86 × 86 mm. It is its own small computer. The face is one slot. The electronics are on the back, in the box.

Must do





Outline 86 × 86 mm. Two unplated 3.8 mm holes on the vertical centre line, 60.3 mm apart, at (43, 12.85) and (43, 73.15) with the origin at the top-left, y downward. M3.5 into the back box. Unplated, so a metal box is not tied to the circuit.



One finger slot between the screws, about 36 × 48 mm, split by a 2 mm gap on the centre line. Left half is brightness. Right half is white warmth, or RGB, depending on mode. Each half is two copper wedges: one wide at the top, one wide at the bottom. Solder mask stays on. A 1–2 mm plain plastic fascia covers them. No metal paint.



Sense slide up, slide down, tap, double-tap, and triple-tap. Direction is the sign of the difference between the two wedges. Absolute finger position is not reported.



Read the wedges on four GPIOs of an ESP32-S3-WROOM-1U-N8 (C2980297), on the back, U.FL toward the right edge. Left bottom = GPIO1 (pad 39). Left top = GPIO7 (pad 7). Right bottom = GPIO2 (pad 38). Right top = GPIO3 (pad 15). No series resistor on these four nets. GPIO0 stays the boot strap and is not a touch pin.



Send the gesture to the control board over Wi-Fi. This board does not connect to the 1×6 and does not carry 24 V.



Make 5 V from mains with an HLK-PM01L, LCSC C19632428, 32 × 18 × 13 mm, 600 mA, 3 kV isolated. It is a finished brick. Live and neutral land on a KF301-5.0-2P screw terminal, LCSC C474881, on the back, opening toward the bottom edge. Pin 1 is live, pin 2 is neutral. Wire up to 1.5 mm².



Fuse the live at 500 mA, time-lag, 250 VAC (JFC2410-0500TS, C136377) before the module. Put a 10D561K varistor (C113236) across the module’s AC pins after the fuse.



Mill a slot through the board between the Hi-Link’s AC pins and its DC pins. Mains copper stays on the terminal side of that slot. DC ground is not bonded to mains, to the terminal, or to the back-box earth.



Regulate 5 V to 3.3 V with an AP2112K-3.3 (C51118). 100 µF on the Hi-Link output, on the Hi-Link side of the diode below.



Program from USB-C (C165948) on the back, opening at the right edge. D− to GPIO19, D+ to GPIO20. Each CC pin 5.1 kΩ to GND. VBUS and the Hi-Link 5 V each pass an SS16 (C84027) before they meet at the LDO input, so a laptop can power the ESP32 and neither 5 V can feed the other.



SW1 shorts EN to GND. SW2 shorts GPIO0 to GND. Both have 10 kΩ to 3.3 V. EN has 1 µF to GND. Flash by holding SW2, tapping SW1, releasing SW2.



Budgets





Hi-Link output: 5 V, 600 mA. The ESP32 transmit spike is about 355 mA at 3.3 V, which the 5 V rail sees as current. A 1 W module is not enough.



Board: 86 × 86 × 1.6 mm. Module height 13 mm. Use a 35 mm back box. 47 mm is comfortable. 25 mm is tight.



Touch: four self-capacitance channels. No solid ground under the wedges.



Isolation: the milled slot is the barrier between mains and the 5 V side. Creepage on the AC side stays at least 3 mm from DC copper on the same layer.



Constraints





2-layer, 1.6 mm, 1 oz, lead-free HASL, tented vias.



All parts on the back. The face is copper wedges and the two screw holes.



The left 24 mm is the mains bay. The wedges start to the right of it so they do not sit over the Hi-Link.



The HLK-PM01L has no LCSC footprint. Holes: AC pins 5.0 mm apart, DC pins 13.4 mm apart, 27.4 mm between the rows, 1.2 mm drills. Confirm those three distances on the module before ordering.



The KF301 library drills were opened to 1.1 mm so the pins fit.



UK switch boxes often have no neutral. This board needs live and neutral. Say so. Do not design around a single live.



Out of scope





A capacitive dropper or any non-isolated mains supply.



A relay, a triac, or any switching of the lighting circuit.



24 V, the strip connector, or the microphone.



USB-C or buttons on the face.



10 kΩ in series with the touch wedges.



Bonding DC ground to earth or to an AC pin.



Eight touch channels. Four GPIOs only. The other GPIOs stay free.





System out of scope





One PCB that combines the control board, the strip, and the wall plate.



A CH32, an RS-485 pair, or an address token on the lighting cable.



Mains on the control board or the strip. The only mains board is the touch panel, and only to power itself.



Driving the lamp by switching its mains. The lamp is the 24 V strip. The plate only sends a gesture.
