[![MCHP](images/microchip.png)](https://www.microchip.com)

# Core Independent Voltage Window Signal Detection Using a Single Comparator

This example presents an alternative method of implementing a voltage window detection (without software core supervision), using a single comparator and the Core Independent Peripherals (CIPs) of a PIC16F176X.


##  Introduction
It is possible to find out whether a measured signal is below or above a certain value/reference using a single comparator. But, what if the desired interval is between two values?
The most convenient and fastest solution is to use two comparators and two references. The results are analyzed to decide which of the three intervals houses the measured signal. 
Using an Analog-to-Digital converter (ADC) and core post-processing will yield the same result, but the process is slower and dependent on core availability. 
This example presents an alternative method of implementing a core independent voltage window signal level detection (without software core supervision), using a single comparator and the Core Independent Peripherals (CIPs) of a PIC16F176X/7X. 
The method is used to implement an Under Voltage Protection (UVP) and Over Voltage Protection (OVP). The representation of the needed voltage window signal level between two thresholds is depicted in the figure below. 
This solution has the advantage of using only CIPs. It does not need core usage, it is considerably faster than an ADC measurement, and it still provides all the configurability advantage for the user.


<br><img src="images/cip-volt-detect-1.png" width="600">

## Detailed Description
The solution presented here uses a single comparator to convert the voltage level into a Pulse-Width Modulated (PWM) signal, with the help of the Programmable Ramp Generator (PRG) CIP as reference. 
The obtained signal is then used as two of the inputs to the Configurable Logic Cell (CLC) CIP configured as a four-input AND-OR that acts as a PWM signal comparator.
The CLC has an output of logic ‘1’, whenever an undervoltage or overvoltage event occurs and an output of logic ‘0’, when the measured signal is in the desired interval. 
The output of the CLC is connected to the Auto-Shutdown (AS) of the Complementary Output Generator (COG) CIP, which in consequence will protect against OV and UV. 
The internal CIP connections are depicted in the figure below and the signals, in the following figure.

<br><img src="images/cip-volt-detect-diagram-1.png" width="600">

<br><img src="images/cip-volt-detect-output-1.png" width="600">

## Software Used
- MPLAB® X IDE 5.30 or newer [(microchip.com/mplab/mplab-x-ide)](http://www.microchip.com/mplab/mplab-x-ide)
- MPLAB® XC8 2.10 or newer compiler [(microchip.com/mplab/compilers)](http://www.microchip.com/mplab/compilers)
- MPLAB® Code Configurator (MCC) 3.95.0 or newer [(microchip.com/mplab/mplab-code-configurator)](https://www.microchip.com/mplab/mplab-code-configurator)

## Hardware Used
- PICDEM Lab II board [DM163046](https://www.microchip.com/developmenttools/ProductDetails/PartNO/DM163046) or a prototyping board
- PIC16F1779 PDIP (any PIC16F176X/7X can be used as well)
- One potentiometer to simulate the variable input voltage, wires, MPLAB® PICkit™ 4 (any other PIC programmer will work)
- One Oscilloscope to verify the signals

## Setup

The following figure depicts the test setup, where VDD and GND are provided by PICkit 4. 
To power the board from the PICkit 4, right click on the current project, followed by Properties/PICkit 4/Option
categories: Power, and check the box ‘Power target circuit from PICkit 4’, then select OK.


<br><img src="images/cip-volt-detect-setup-1.png" width="600">


### PIC MCU Configurations

- System module: System clock select – FOSC, Internal Clock – 8 MHz_HF, PLL Enable

### CIPs used

- TMR2: Clock source – FOSC/4, time period – 2 us (500 kHz switching frequency) – decides the SMPS switching frequency.
- COG1: Mode – Half-Bridge; Clock source – FOSC; COGA PIN Steering – waveform; COGB PIN
Steering – waveform; Rising event PWM3 – level trigger; Falling event PWM3 – level trigger; PWM3
low-to-high transition triggers the rising event of the COG and the high–to–low transition triggers the
falling event, the dead-band delay set in the demo is 812.5 ns but this is the value that is used to
control the high threshold and the low threshold, so the value must be adjusted according to the
user’s needs.
- FVR: FVR_buffer1 and FVR_buffer2 – 4x; 4.096V is used as the PRG source.
- PRG: Ramp generating mode – alternating ramp generator; Voltage input source – FVR_buffer1;
Slope rate – 2.5V/us; Ramp rising timing source – PWM3_output/level sensitive/active_high, Ramp
falling timing source – PWM3_output/Level sensitive/active_low, PRG is configured as alternating
ramp generator; Slope rate: 2.5V/us.
- PWM3: Select timer – Timer2, duty cycle 50%, used as start ramp rising, start ramp falling for PRG,
and as a signal source for the COG.
- CMP1: Positive input – PRG1; Negative input – CIN4, the negative input is connected to the
equivalent of the input voltage or the measured signal, and the positive input is connected the PRG
output. The output of the comparator will represent a PWM signal with the duty cycle equivalent and
proportional to the measured voltage level.
- CLC1: Mode – AND–OR; AND1 Input is C1_OUT negated and COG1A; AND2 input is C1_OUT and
COG1B. The output of the CLC can be connected directly to the auto-shutdown of the COG used for
the SMPS loop (it is advised that you check the COG auto-shutdown tab in MCC to see which CLC is
accepted as input). To stop the device when there is irregular input, an interrupt routine can be
implemented to deal with the system during protection.
- OPA1: Channel select: positive channel – PRG1_OUT and negative channel – ‘anything’ because is
disabled; Set as ‘Unity gain’, this is used to monitor the internal PRG signal with the oscilloscope.

The following figure depicts the MCC peripherals used, while the last figure shows the CLC connections made in MCC.
The lines of code added ‘while (!PRG1_IsReady()); PRG1_StartRampGeneration();’ allow
the PRG time to initialize and start when it is ready.


<br><img src="images/cip-volt-detect-mcc-1.png" width="600">

<br><img src="images/cip-volt-detect-mcc-2.png" width="600">


# Results
The following results were obtained using the oscilloscope to measure the required signals found at the pins depicted above. The results prove the correct functionality of the solution.
The following figure depicts the comparator output resulting from the comparison between the input
voltage (sampled voltage) and the PRG ramp (sampling signal).


<br><img src="images/cip-volt-detect-output-2.png" width="600">


The following figure depicts the CLC1 output, when the measured input voltage is in the desired voltage window. For the SMPS application, this is equivalent with a safe input voltage operation.


<br><img src="images/cip-volt-detect-output-3.png" width="600">


The following figure depicts the CLC1 output, when the measured input voltage is above the desired
voltage window. For the SMPS application, this is equivalent with an input over voltage detection.


<br><img src="images/cip-volt-detect-output-4.png" width="600">


The following figure depicts the CLC1 output, when the measured input voltage is below the desired
voltage window. For the SMPS application, this is equivalent with an input under voltage detection.


<br><img src="images/cip-volt-detect-output-5.png" width="600">


## Conclusion
The solution presented in this example solves the problem of voltage window signal detection by using a single comparator. 
The measured signal is converted into a PWM signal, where the duty cycle is equivalent to the voltage level and the threshold detection is implemented by the PWM signal comparison with logic cells.
The resulting function is faster than the ADC approach and does not need core supervision during the
operation. It offers threshold configuration, change, and sampling speed selection.
The practical demonstration envisages a UVP and OVP example. This approach proves that the UV and OV protection function that usually needs two comparators and two voltage references can be
implemented using a single comparator, if the designer transitions the comparison between four voltage signals into a comparison between four PWM signals.

