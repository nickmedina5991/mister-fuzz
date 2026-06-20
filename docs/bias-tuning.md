# Bias Tuning Procedure

After assembly, the 10kΩ trim pot (RV3) must be adjusted to set Q2's 
collector voltage to 4.5V.

## Equipment needed
- Digital multimeter

## Procedure
1. Connect power.
2. Set the DMM to DC volts.
3. Connect the DMM black lead to GND (sleeve of any audio jack).
4. Probe Q2's collector. (Marked on silkscreen, or bottom pin of Q2.)
5. Read the voltage. Likely starting value: 2-6V depending on the 
   trim pot's initial position and your specific 2N3904 hFE.
6. Turn the trim pot screw clockwise to decrease the voltage, 
   counterclockwise to increase. The 3362P is a single-turn trimmer 
   so changes happen quickly.
7. Adjust until you read 4.5V +/- 0.1V.
8. Power down, plug in, and play. Done.

## Why 4.5V?
Q2's collector swings around this DC bias point. At 4.5V the transistor 
has maximum headroom both up (toward +9V) and down (toward 0V). This 
gives the Fuzz Face its characteristic symmetric-to-asymmetric clipping 
behavior as input level increases.

## Troubleshooting
- If voltage doesn't change as you turn the trimmer: check trimmer 
  orientation, verify pin 2 (wiper) goes to Q2 collector.
- If voltage is stuck near 0V: Q2 is saturated. Check Q1 hFE and 
  consider swapping it for a lower-gain unit.
- If voltage is stuck near 9V: Q2 is cut off. Check transistor 
  pinout (E-B-C correct).