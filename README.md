# Open Nanogrid
Drafting an open standard for an extra low voltage DC electricity grid.

Most small devices use low voltage DC. Let's create a DC grid that can feed on renewable energies (RE), doesn't need to be tied to the AC grid and is safe to develop with according to Separated Extra Low Voltage (SELV) standards. This is not about another war on currents, AC has it's place for large appliances and the present infrastructure is a well tuned system that was built over 100 years. It's also not about a micro grid where a whole neighbourhood is connected with AC. The Open Nanogrid is focussed on sourcing and consuming energy on a smaller level, from your roof into your living room, from the river into your hackspace, from SunZilla into your Fablab's CNC mill - all with a safe voltage for short distances.

![AC vs DC Facts](/img/Open-Nanogrid-Layout.png)

Here's a 5 min introduction video I made for the Shuttleworth Foundation: https://www.youtube.com/watch?v=FQdyD6pwzek

This is a random stream of thoughts, feel free to add yours too!

- Any voltage over 50 VAC or 120 VDC is not safe to the touch
- It is expensive to generate a true sine wave alternating current out of low voltage DC from renewable energies
- Solar panels output low voltage DC
- TEG (thermo electric generators) output low voltage DC
- Small permanent magnet alternators used in a generator at a wind turbine or a water turbine usually operate at below 100V


Only a few systems really do need the high voltage AC of 230V or 110V, like electric ovens and heaters.

- Laptops need about 12-20 VDC
- Flat screens run on DC, especially if they have an LED backlight.
- Mobile phones, smartphones, tablet PCs etc are all charged and run with 5VDC
- LEDs have a low forward voltage of less than 5V. However, there are also COB LEDs with lots of emitters in series.
- Desktop Computers (although inefficient compared to laptops and smartphones) do need 12 VDC, some also 5 VDC and 3,3 VDC.

Difficulties and negative points about a low voltage DC grid:

- Grid size is smaller with a lower voltage or:
- available Power is lower at the same AWG / cable dimensions, as the Joule effect will heat up cables faster with a lower voltage.
- special connectors have to be chosen, there is no good standard
- It is inefficient for large consumers


Best of both worlds: A hybrid 230 VAC + Extra Low Voltage (ELV) DC infrastructure; 230VAC for high power appliances like heavy machinery, ovens, ELV DC for any low power consumer electronics, efficient lighting, small scale renewable energy sourcing and experiments.

## Voltage

Maybe a multi phase system?

- N: 0V, GND
- L1: nominal 12VDC, (10-15VDC ?)
- L2: nominal 48VDC, (24-52VDC ?)

For increased efficiency with high loads and long cables a higher voltage would be better. 100V MOSFETs are commonly available, so a 84V (equals 20 LiPo cells in series) could be a better alternative.

84 VDC is plug and play compatible with some traditional AC SMPS (Switch Mode Power Supplies). However, no integrated DC-DC converter solutions are available for such high voltages and decent current ratings. External MOSFET and additional parts are needed -> more PCB space, part count, higher cost.

Cheap 2x2.5-10mm² (1 Phase) or 4x2.5-10mm² (2 Phase) cabling may be used for branches of the grid, no heavy or double insulation is needed. Loudspeaker cables can be sufficient. The voltage drop in the wires (Joule effect heating up the cable) should be less  than 10%.

Power buffering with batteries;

Example 1: A nominal voltage of 3x12V = 36V equals three lead acid batteries connected in series. If fully charged, grid voltage at the source would be 3x14V = 42V, at empty batteries about 32V. The voltage level can be sanitized with a SEPIC converter.

Example 2: A voltage of 48V could consist of 12 LiPo cells in series. 50.4V when fully charged, 40V when empty. A DC/DC step-up converter could keep the voltage steady at 50V at all times and shut down when the SOC (state of charge) of the batteries is too low.

An 48V grid voltage may be compatible with PoE, Power over Ethernet 802.3af (802.3at Type 1) and 802.3at Type 2. There are passive PoE systems that use 12..48VDC. Many network devices (switches, phones, access points, routers, cameras)  are powered via PoE.

A nanogrid could be on/off grid redundant with one efficient (ATX gold?) power supply and an active diode (instead of two Schottky diodes). It is easier and more efficient to switch DC synchronous vs AC synchronous, because one does not have to establish a phase lock to get the waveforms in sync and cheaper low voltage MOSFETs can be used.

## Smart Grid

The start would be to equip an energy monitoring system, like Open Energy Monitor (OEM). Communication could take place via
- Wireless connections (even to non-grid-tied appliances. NRF24L01+, RFM12, ESP8266, BT,..)
- Wired LAN connection (power hungry, expensive and uses additional cabling)
- PLC (Powerline Communication) with an Open Source protocol and hardware design.

Research: What about RS485 between two phases, would that be possible? Is a choke / low-pass needed at the low impedance energy sources and appliances? What frequency is used best for transmitting data via a modulated power line? Coupling via passive RC-highpass?
There might be an integrated circuit for low voltage DC PLC communication (I didn't find one).

### Intelligence, active electronics

A controller is monitoring some or all grid parameters, can be connected with a BMS (battery monitoring/managment system) or may even contain BMS functions. 

Interesting parameters:
- l1 phase current (has got to be signed for bidirectional measurement)
- l2 phase current (has got to be signed for bidirectional measurement)
- voltages at source points and end points (insights to calculate losses)

- battery bank temperature sensors
- over current (OC) protection
- over voltage (OV) and under voltage (UV) protection
- Uninterruptable Power Source function
- pure solid state switches, MOSFET/SSR<ref>Solid State Relay</ref> usage, no conventional electromechanical relays.

- algorithms to switch on chargers or grid-tied SMPS (switch mode power supplies) if source impedance gets to high/grid voltage too low.

- digital zero voltage diode (ZVD) function via comparator+ISR at ADC or interrupt attached to a plain digital port pin. SSR could then be used as OC protection, UV protection and ZVD.   

## Hardware

Voltage sensing should be through passive voltage dividers with appropriate headroom and a high impedance connection to the ADC, we don't need the galvanic isolation at these low voltages.

Current sensing should not be shunt based for high powers, but rather with a hall effect sensor or inductive. At DC, inductive sensing is not possible I guess, so we have to stick to hall sensors. MOSFET Rds parameters yield a Vds that can be sampled with a high gain differential amplifier.

The integrated current sensor packages from Allegro are quite expensive. The following will be suitable for a 200A branch.
Bidirectional integrated hall effect current sensor:
ACS759	±50A to 200A
ACS756	±50A to 100A

Small OLED display for easy overview at the controller, showing the momentary power consumption and accumulated energy, grid voltage, ...

### Cabling

If the infrastructure is build from scratch and completely new, then it's best to start with a hybrid AC/DC (Alternating Current / Direct Current, not the famous rockband) grid: 

cabling:
- open or closed ring topology with branches, e.g. 3p AC household EIS: 1 x NYM-J 5G2.5, laid in parallel with 4x6..16 for ELVDC.

Important: How to distinguish the cables other than by their inner topology? ELVDC cables should be marked differently.


### Connectors, Sockets, Plugs, Terminals

Suggestions for sockets and plugs:

- large open screw terminals for high current use cases (which is somewhat legal, because the LVDC grid is safe to the touch )
- PowerCon sockets (expensive, proprietary, e.g. by Neutrik. How many poles?)
- SpeakOn sockets (moderately expensive, well suited because they carry 4 poles and are aimed for moderately high currents)
- XLR sockets (rather inexpensive, great for small appliances like phone chargers, laptops, screens, up to 16A per pin)
- Custom Open Source snap/screw terminals with M6 screws, 3D printed/milled/turned.
- RC battery plugs (bad UX, not made for many mating cycles, best suited for more permanent installation)
- USB3 type C (needs active electronics for higher power levels, only 5A x 20V = 100W)
- Wieland DMX IP RST20i3S 50V/20A (for outdoor interconnections, expensive at $10)
- Anderson PowerPole for batteries and large interconnections

- A new, open source connector? Check https://github.com/OpenNanogrid/ON-socket for more ideas.

Any other recommendations?

CEE sockets are way too expensive, overkill and not meant for such low voltages. They may also be confused with the mains grid.

Fuses, circuit breakers, controllers, BMS, PDUs and so on are encased in a separate enclosure and are not in the same  mains breaker box.

XLR specifications from Neutrik NC3-FX

 > Capacitance between contacts ≤ 4 pF
 Contact resistance ≤ 3 mΩ (inner)
 Dielectric strength 1,5 kVdc
 Insulation resistance  2 GΩ (initial)
 Rated current per contact 16 A
 Rated voltage 50 V
 
## Separated or safety extra-low voltage ([SELV](http://en.wikipedia.org/wiki/Extra-low_voltage))

IEC defines a SELV system as "an electrical system in which the voltage cannot exceed ELV under normal conditions, and under single-fault conditions, ''including'' earth faults in other circuits".

There exists some confusion regarding the origin of the acronym: "SELV" stands for "''separated'' extra-low voltage" in installation standards (e.g., BS 7671) and for "''safety'' extra-low voltage" in appliance standards (e.g., BS EN 60335).

A SELV circuit must have:
- protective-separation (i.e., double insulation, reinforced insulation or protective screening) from all circuits other than SELV and PELV (i.e., all circuits that might carry higher voltages) As an example, in New Zealand, ELV wiring in domestic premises must be installed at a minimum distance of 50 mm from low voltage wiring or have a separate insulating barrier such as conduit. See http://en.wikipedia.org/wiki/Extra-low_voltage
- simple separation from other SELV systems, from PELV systems and from earth (ground).

The safety of a SELV circuit is provided by
- the extra-low voltage
- the low risk of accidental contact with a higher voltage;
- the lack of a return path through earth (ground) that electric current could take in case of contact with a human body.

The electrical connectors of SELV circuits should be designed such that they do not mate with connectors commonly used for non-SELV circuits.
