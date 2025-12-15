# Final report for GSoC'25

My GSoC Experience with Analog Devices  
<img src="https://github.com/MarileneGarcia/marilene.github.io/blob/main/docs/GSoC/midia/setup_test.jpg?raw=true" alt="setup" width="470" height="700">

## Table of Contents
1. [About Me](#about-me)
2. [About Analog Devices](#about-analog-devices)
3. [Project Overview](#project-overview)
4. [Work Done](#work-done)
5. [Important Links](#important-links)
6. [Future Work](#future-work)
7. [Thanks to My Mentors](#thanks-to-my-mentors)

## About Me
My name is Marilene Andrade Garcia. I am a computer engineer from Brazil. Since finishing my degree, I have been seeking opportunities to work in the embedded development field.

I have been working with Analog Devices under a Linux Foundation project for the past three months, writing a Linux Kernel device driver in the Industrial Input/Output (IIO) subsystem for an analog-to-digital converter (ADC).

## About Analog Devices
Analog Devices, Inc. (ADI) is an American multinational semiconductor company specializing in data conversion, signal processing, and power management technologies.

## Project Overview
When I applied for the program, the initial goal of the project was to provide support for ADE9113 chips. However, since two mentees were accepted, my mentors decided to adjust the scope and assign one device to each mentee.

I was assigned the task of providing support for the __MAX14001/MAX14002__, configurable and isolated 10-bit ADCs for multi-range binary inputs.

In addition to ADC readings, the MAX14001/MAX14002 offers several features, such as a binary comparator, filtered readings that can provide the average of the last 2, 4, or 8 ADC values, and an inrush comparator that detects inrush current. There is also a fault detection feature capable of diagnosing seven possible fault conditions, as well as an option to select either an external or internal ADC voltage reference.

## Work Done
### Assembling the Hardware Setup
- To develop and test the code, my mentors sent me a Raspberry Pi 5, and an evaluation board MAX14001PMB which contains two MAX14001 devices: one for measuring voltage and the other for measuring current.
- I used a USB-to-Serial FTDI232 adapter to access the Raspberry Pi terminal from my computer.
- I also configured my LAN to allow access to the Raspberry Pi via Wi-Fi and SSH.
- Since I do not yet have a DC source, I used the Raspberry Pi’s 5V power supply and a 10K potentiometer to test voltage measurements. Unfortunately, the current supply was too low, so the current measurements were unreliable.

### Studying the Evaluation Board
- The MAX14001PMB evaluation board can measure high line voltage values (up to 230V AC or ±325V DC) and load current values (up to 5A). However, the MAX14001 only accepts input values below the Vrefin, which is usually 1.25V. To handle this, the board includes an circuitry that scales the signals proportionally and applies an offset, allowing the measurement of negative values.
- To analyze the output readings, I studied the evaluation board circuitry and derived formulas to calculate the input current and voltage. These formulas are still under review to verify their accuracy and can be found in the following PDF: [MAX14001PMB_circuit_analysis](https://github.com/MarileneGarcia/marilene.github.io/blob/main/docs/GSoC/MAX14001PMB_circuit_analysis.pdf)

### Studying the MAX14001/MAX14002 and Developing the Driver Code
- To write the driver code, I studied the MAX14001/MAX14002 datasheet, as well as the IIO subsystem and other IIO ADC drivers.
- I wrote several versions of the code, but my mentor and I decided to prepare a clean and organized version for upstream submission. This version currently supports only the features related to reading two registers: one containing the latest ADC reading and another containing the latest filtered ADC reading. The additional features are planned to be submitted upstream in future patches.
- All of the work done can be followed in this PR. It includes the driver code, the dt-binding documentation, and an overlay to include MAX14001 into raspberry pi dts.
[iio: adc: Add initial driver support for MAX14001/MAX14002](https://github.com/analogdevicesinc/linux/pull/2848)

### Developing an User Space Code to use the Driver
- To test the driver and get readings from the MAX14001PMB, the following user-space code was developed: [MAX14001PMB Reader Program](https://github.com/MarileneGarcia/max14001pmb_reader)

### Upstreaming the Device Driver Kernel Code
- I generated three patches based on the [IIO tree, testing branch](https://git.kernel.org/pub/scm/linux/kernel/git/jic23/iio.git/log/?h=testing) to submit the driver code upstream: one with the cover letter explaining the driver, one with the device tree binding documentation, and another with the driver code.
- After sending the patches, we discovered a commit set from 2023 with the same goal of providing support for the MAX14001 device, which had not yet been accepted. It is therefore likely that my changes will be merged into the previously existing code. I will continue by generating updated versions of the older commits to improve the code until it is accepted. All related discussions can be found here: [lore kernel max14001](https://lore.kernel.org/all/?q=max14001)

### Demo of the user-space program on Raspberry Pi 5 reading values from my MAX14001 driver version
- [Demo Linux Kernel Driver MAX14001](https://www.youtube.com/shorts/xqOkkvufINA)  
![demo](https://github.com/MarileneGarcia/marilene.github.io/blob/main/docs/GSoC/midia/demo.gif?raw=true)

## Important Links
- Pull Request with the MAX14001 driver code in the Analog Devices repository: [https://github.com/analogdevicesinc/linux/pull/2848](https://github.com/analogdevicesinc/linux/pull/2848)
- My fork of the Analog Devices repository with the MAX14001 driver code: [https://github.com/MarileneGarcia/linux/tree/rpi-6.6.y_GSOC_2025_MAX14001](https://github.com/MarileneGarcia/linux/tree/rpi-6.6.y_GSOC_2025_MAX14001)
- My repository of the user-space MAX14001PMB reader program: [https://github.com/MarileneGarcia/max14001pmb_reader](https://github.com/MarileneGarcia/max14001pmb_reader)
- MAX14001PMB circuit formulas analysis: [https://github.com/MarileneGarcia/marilene.github.io/blob/main/docs/GSoC/MAX14001PMB_circuit_analysis.pdf](https://github.com/MarileneGarcia/marilene.github.io/blob/main/docs/GSoC/MAX14001PMB_circuit_analysis.pdf)
- Kernel mailing list thread related to MAX14001: [https://lore.kernel.org/all/?q=max14001](https://lore.kernel.org/all/?q=max14001)
- Kernel commits integrated into version 6.19-rc1:  
[dt-bindings: iio: adc: add max14001](https://git.kernel.org/pub/scm/linux/kernel/git/torvalds/linux.git/commit/?h=v6.19-rc1&id=192e5bbf0a8d7a629e6f9fa9e2fae54c3268bb7f)  
[iio: adc: max14001: New driver](https://git.kernel.org/pub/scm/linux/kernel/git/torvalds/linux.git/commit/?h=v6.19-rc1&id=59795109fa67d2bee7c8c2847a487d4dddb428c1)

## Future Work
- Submit additional patches with the features missing in the first driver version, such as support for the binary comparator, inrush current, and fault alert. I intend to continue sending patches to cover all the device’s features even after GSoC ends.
- Review the formulas used in the user-space program to ensure their accuracy.
- Acquire a DC source to test higher voltage and current inputs.

## Communication with the Organization and Mentors
We held weekly Zoom meetings and maintained an open chat on Slack.

## Thanks to My Mentors
I would like to thank my mentors Marcelo Schmitt, Ceclan Dumitru, Jonathan Santos, and Dragos Bogdan for their guidance, code reviews, and explanations about the IIO subsystem. Special thanks to Marcelo, with whom I have communicated almost daily for reviews and advice, and who conducted most of the driver code review. I also want to thank Dumitru for providing valuable insights at the beginning of the development process.