# Design Notes

## The Fuzz Face circuit

The Fuzz Face is a two-transistor common-emitter amplifier with global negative feedback. The signal path is short and conceptually simple:

- Q1 is the input stage. It receives the AC-coupled guitar signal at its base and provides a modest amount of voltage gain. Its emitter is grounded.
- Q2 is the output stage. Its base is direct-coupled to Q1's collector (no AC coupling between the stages), and it provides the bulk of the circuit's voltage gain. Its emitter goes through the fuzz pot to ground, with an electrolytic capacitor (C3) bypassing part of the emitter resistance to set the AC gain.
- A 100kΩ feedback resistor (R4) connects Q2's emitter back to Q1's base. This sets the DC operating point of the entire circuit and creates the global feedback loop responsible for the Fuzz Face's distinctive interaction with input level and impedance.
- The output is taken from a voltage divider between Q2's collector and the supply, formed by R2 (470Ω) and the bias trimmer (originally 8.2kΩ, replaced here with a 10kΩ adjustable trim pot that should begin tuning around 6kΩ).

The whole circuit produces its characteristic sound from a combination of high gain (around 60dB at maximum fuzz), asymmetric clipping at the rails, and the way the bias point shifts during saturation. I will be comparing the silicon circuit with Electrosmash's published analysis of the germanium fuzz face.

## Why silicon instead of germanium

The original 1966 Fuzz Face used germanium PNP transistors such as the AC128, NKT275, or OC75. These parts have characteristics that are difficult to replicate today:

- Low and inconsistent gain (hFE typically 50 to 100, with massive part-to-part variation)
- High leakage current that drifts with temperature
- Soft, gradual turn-on behavior at low Vbe
- Limited production today, with surviving stock often expensive and unreliable

Modern 2N3904 NPN silicon transistors are several orders of magnitude cheaper, available everywhere, far more consistent, and electrically more stable across temperature. I used these because I had them in stock. The tradeoff is: silicon Fuzz Faces are documented as having slightly less bass response, sharper top end, and less of the "vocal" midrange character that germanium versions are known for. The substitution preserves the essential Fuzz Face behavior (high gain, asymmetric clipping, dynamic interaction with guitar volume) while accepting a different voicing.

The silicon version is also a real, recognized variant in pedal history. Dallas Arbiter's later Fuzz Face production runs used BC108 silicon transistors, and that lineage continues through pedals like the Roger Mayer Axis Face and various boutique silicon Fuzz Face clones today.

## Bias point derivation

The original Fuzz Face uses an 8.2kΩ fixed resistor in series with the 470Ω collector resistor on Q2. With germanium transistors of hFE around 70 to 100, this lands Q2's collector at approximately 4.5V, which is the target operating point. With modern 2N3904s (hFE typically 150 to 300), the circuit biases differently and the 8.2kΩ value does not produce the correct operating point.

The LTSpice simulation included a parametric sweep of this resistor from 4kΩ to 20kΩ, with Q2's collector voltage measured at each value. The sweep showed that a value of approximately 6kΩ landed Q2's collector at the target 4.5V with LTSpice's generic 2N3904 model. The production design uses a 10kΩ trim pot in this position so that each build can be tuned to the specific transistors installed, since real-world 2N3904 hFE varies considerably between units.

The remaining simulation bias points settled at:

- Q1 collector: 1.34V
- Q1 base: 0.62V
- Q2 base: 1.34V (direct-coupled to Q1 collector)
- Q2 emitter: 0.69V
- Q2 collector: 4.53V (the target)

Note that the Q1 collector voltage of 1.34V is intentionally low. This is not a mistake. The Fuzz Face deliberately biases Q1 close to its lower rail so that the positive half of the input signal has a large amount of headroom while the negative half clips early. This asymmetric biasing is the source of the Fuzz Face's signature asymmetric clipping behavior. A symmetrically biased two-stage amplifier would produce a much more "square" distortion that lacks the Fuzz Face's musical character.

These bias values match the published ElectroSmash analysis for the Fuzz Face within the expected range, confirming that the silicon variant preserves the topology's intended operating behavior.

## Simulation results and analysis

Four key analyses were run in LTSpice to verify circuit behavior before committing to the PCB. The plots are stored in `simulation/plots/`.

### Bias resistor sweep

A DC operating-point sweep of the bias resistor R3 from 2kΩ to 10kΩ in 2kΩ steps, with Q2's collector voltage measured at each value. The sweep identified 6kΩ as the value that produces the target 4.5V bias point with the LTSpice 2N3904 model. This value becomes the starting point for the physical trim pot adjustment during board assembly.

### Transient response

A time-domain simulation with a 200mV 440Hz sine input (representing a moderately played guitar note at A4) showed the output waveform clipping hard at approximately +480mV on positive peaks and around -100mV on negative peaks. This asymmetric clipping behavior is the audible signature of the Fuzz Face. The recovery curve between cycles ("shark fin" shape) comes from the AC coupling capacitors recharging after saturation, which contributes to the circuit's characteristic compression and attack response.

### Fuzz pot sweep

A parametric sweep of the fuzz pot position from 0.01 to 0.99 (corresponding to the wiper position from full ground to full Q2 emitter connection) with the input held at 1mV to keep the lowest fuzz setting linear. The output traces show:

- At minimum fuzz: small clean sine, low amplitude
- At quarter fuzz: gentle soft clipping, modest amplitude
- At half fuzz: pronounced clipping with rounded peaks
- At three-quarter fuzz: heavy clipping approaching square wave
- At maximum fuzz: full square wave saturation

The 20dB range between minimum and maximum fuzz settings matches the published ElectroSmash measurements for the original germanium circuit. At realistic guitar signal levels of 100mV to 300mV, even the lowest fuzz settings already saturate the circuit, which is why the Fuzz Face is famous for cleaning up dramatically when the guitar's volume knob is rolled back. The pedal effectively becomes a clean booster at low input levels.

### AC frequency response

A small-signal AC sweep from 10Hz to 100kHz at five fuzz pot positions revealed that the fuzz control does more than change gain. It actively reshapes the frequency response:

- Peak passband gain: approximately 62dB at maximum fuzz
- Low-frequency corner: approximately 50 to 80Hz (depending on fuzz setting)
- High-frequency corner: approximately 10kHz
- Midrange resonant peak: 115Hz at maximum fuzz, flattening at lower fuzz settings
- Total gain change across the fuzz control range: approximately 20dB

The resonant peak at 115Hz sits right in the fundamental range of the guitar's low strings (low E at 82Hz, A at 110Hz), contributing to the chunky low-mid emphasis that fuzz pedals are known for. At low fuzz settings the response flattens out and the circuit behaves more like a wideband clean amplifier.

### Comparison to published reference

| Property | Simulation | ElectroSmash (germanium) | Match |
|----------|------------|--------------------------|-------|
| Max passband gain | ~62dB | ~58dB | Close |
| Low corner | ~50 to 80Hz | 14Hz | Higher (expected for silicon) |
| High corner | ~10kHz | not specified | N/A |
| Midrange peak | 115Hz | 100 to 200Hz region | Match |
| Gain change with fuzz pot | ~20dB | ~20dB | Match |

The higher low-frequency corner in the silicon simulation compared to the germanium reference is expected behavior. Silicon Fuzz Faces are documented as having slightly less bass response than germanium versions, and the simulation captures this difference correctly. The overall agreement with published data confirms the silicon substitution preserves the circuit's essential frequency-domain behavior. Silicon offers higher gain and more aggressive tone, which some people prefer.

## Breadboard verification

After simulation, the circuit was prototyped on a solderless breadboard using parts from stock. Q2's collector voltage was measured at the breadboard and adjusted via the trim pot to land at 4.5V, matching the simulated target. Real-world waveforms were verified against simulation using an oscilloscope, with the asymmetric clipping behavior visible at moderate input levels and the expected square wave saturation at higher inputs. The breadboard prototype validated the bias point, the transient behavior, and the basic frequency response before moving to PCB layout.

## Power supply design

The pedal accepts power from either a 9V battery or a 2.1mm DC barrel jack adapter, with several design choices made to handle the realities of pedalboard use:

### Battery and DC jack switching

The DC barrel jack is a switched type. When no DC adapter is plugged in, an internal switch connects the battery positive terminal to the +9V rail. When a DC adapter is inserted, the switch breaks this connection and the adapter powers the circuit instead, leaving the battery electrically isolated. This is the standard pedal-design trick to prevent the battery from being drained when an external supply is used.

### TRS input jack as battery disconnect

The input audio jack is wired as a stereo (TRS) jack with a mono (TS) guitar cable in mind. The ring contact is connected to the battery negative terminal. When no cable is plugged in, the ring is floating and the battery's path to circuit ground is broken, so no current flows. When the guitar cable is inserted, the cable's sleeve shorts the ring to the sleeve contact, completing the battery's path to ground. This means that unplugging the guitar cable physically disconnects the battery, preserving its life when the pedal is not in use.

### Reverse-polarity protection

A 1N4001 silicon rectifier diode (D2) sits in series with the DC jack's positive supply path. Boss-standard pedal power is center-negative, but plugging in a center-positive adapter by mistake is a way to kill this pedal instantly (the transistors do not survive reversed polarity). The diode blocks reverse current at the cost of a small forward voltage drop of about 0.7V, which is acceptable for this circuit since it does not require precise supply voltage.

### Supply decoupling

A 47µF electrolytic capacitor (C4) is placed across the +9V rail near the DC input. This filters out switching noise from pedal power adapters and provides a local energy reservoir to handle transient current demands during signal peaks. Audio circuits with high gain (like this one at 60dB) are sensitive to power supply noise, since any ripple on the supply rail couples directly into the output. The decoupling cap suppresses this.

## PCB layout decisions

The board is a two-layer through-hole design produced in KiCad. Several deliberate choices were made:

### Two-layer with stitched ground pour

Both copper layers carry a ground pour connected through vias. This creates a low-impedance return path for the signal currents and acts as a shield against external interference. For a circuit with this much gain, single-sided ground routing would create ground loops and pick up hum. The stitched ground plane is one of the most impactful changes a PCB layout can make for a high-gain analog circuit.

### Edge-mounted pots and jacks

The two 16mm potentiometers (fuzz and volume) and the 1/4-inch audio jacks are placed along the board edges so that their shafts and barrels poke through corresponding holes enclosure. This eliminates the need for off-board wires running to panel-mounted controls, which would otherwise pick up noise.

### Short Q1 input loop

The input coupling capacitor (C1) and Q1's base are placed close together with a short trace between them. Q1 is the input stage, so any noise picked up before this point gets multiplied by the full circuit gain downstream (around 60dB, or 1000 times). Keeping this loop short minimizes the antenna area that could pick up hum, radio interference, or coupling from other parts of the circuit.

### Silkscreen documentation

The board's silkscreen layer labels every component (R1 through R4, C1 through C4, Q1 and Q2, RV1 through RV3), all major pin connections (LED-, IN, OUT, DC, 9V-GND, Bias), and includes custom branding with a logo.

## Mechanical design notes

The board's outline is sized to fit a standard Hammond 1590B enclosure (or equivalent). Mounting holes are placed at the corners to accept M3 screws into PCB standoffs. The two 16mm pots and the audio jacks align with hole positions on the enclosure's top and side panels respectively.

The Fusion 360 enclosure model (in `enclosure/`, coming once that work is complete) will include the precise drill template for these holes, the pot shaft cutouts, the audio jack threaded mounts, the DC jack hole, the 3PDT footswitch hole, and the LED bezel hole.

## Bill of materials

See `hardware/bom.csv` for the full parts list with supplier links and order quantities.

Key parts:

- Q1, Q2: 2N3904 NPN BJT
- D1: 5mm diffused LED (color to taste)
- D2: 1N4001 silicon rectifier diode
- R3 (bias trimmer): Bourns 3362P 10kΩ multi-turn trim pot
- RV1 (fuzz): Alpha 16mm 1kΩ linear panel-mount pot
- RV2 (volume): Alpha 16mm 500kΩ audio-taper panel-mount pot
- Audio jacks: StompBoxParts JACK-1024 (TS input), JACK-1025 (TRS output)
- DC jack: Panel Mount 2.1mm switched DC barrel jack
- 3PDT footswitch: StompBoxParts pro latching solder-lug type

## References

- ElectroSmash, "Fuzz Face Analysis": https://www.electrosmash.com/fuzz-face
- R.G. Keen, "The Technology of the Fuzz Face": http://www.geofex.com/article_folders/fuzzface/fftech.htm
- Barbarach's website and tutorials on designing and building a Fuzz Face clone (https://barbarach.com/articles/building-a-fuzz-face-clone/)
- Original Dallas Arbiter Fuzz Face schematic (1966), various reproductions available online