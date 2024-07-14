### Introduction to UART Protocols

The UART (Universal Asynchronous Receiver-Transmitter) is a fundamental communication protocol used in device-to-device communication. It facilitates serial communication, where data is transmitted bit by bit over a single line. In two-way communication, UART uses two wires: one for transmitting data (Tx) and one for receiving data (Rx). This simplicity reduces the need for extensive circuitry and wiring, making UART an efficient and cost-effective solution.


![05_Understanding-UART_01_w1280_hX](https://github.com/user-attachments/assets/150ecc63-c3e9-46f1-b975-1ed357e42c29)

Where is UART used?
-------------------

UART was one of the earliest serial protocols. The once ubiquitous serial ports are almost always UART-based, and devices using RS-232 interfaces, external modems, etc. are common examples of where UART is used.\
In recent years, the popularity of UART has decreased: protocols like SPI and I2C have been replacing UART between chips and components. Instead of communicating over a serial port, most modern computers and peripherals now use technologies like Ethernet and USB. However, UART is still used for lower-speed and lower-throughput applications, because it is very simple, low-cost and easy to implement.


Timing and synchronization of UART protocols
--------------------------------------------

One of the big advantages of UART is that it is asynchronous -- the transmitter and receiver do not share a common clock signal. Although this greatly simplifies the protocol, it does place certain requirements on the transmitter and receiver. Since they do not share a clock, both ends must transmit at the same, pre-arranged speed in order to have the same bit timing. The most common UART baud rates in use today are 4800, 9600, 19.2K, 57.6K, and 115.2K. In addition to having the same baud rate, both sides of a UART connection also must use the same frame structure and parameters. The best way to understand this is to look at a UART frame.


![2024-07-14 12_41_15-Understanding UART _ Rohde   Schwarz - Brave](https://github.com/user-attachments/assets/7d7baa64-5eae-4582-aefc-eae335ffe6c6)

As with most digital systems, a "high" voltage level is used to indicate a logical "1" and a "low" voltage level is used to indicate a logical "0". Since the UART protocol doesn't define specific voltages or voltage ranges for these levels, sometimes high is also called "mark" while low is called "space". Note that in the idle state (where no data is being transmitted), the line is held high. This allows an easy detection of a damaged line or transmitter.

### Start and stop bits

Because UART is asynchronous, the transmitter needs to signal that data bits are coming. This is accomplished by using the start bit. The start bit is a transition from the idle high state to a low state, and immediately followed by user data bits.\
After the data bits are finished, the stop bit indicates the end of user data. The stop bit is either a transition back to the high or idle state or remaining at the high state for an additional bit time. A second (optional) stop bit can be configured, usually to give the receiver time to get ready for the next frame, but this is uncommon in practice.

### Data bits

The data bits are the user data or "useful" bits and come immediately after the start bit. There can be 5 to 9 user data bits, although 7 or 8 bits is most common. These data bits are usually transmitted with the least significant bit first.

Example:\
If we want to send the capital letter "S" in 7-bit ASCII, the bit sequence is 1 0 1 0 0 1 1. We first reverse the order of the bits to put them in least significant bit order, that is 1 1 0 0 1 0 1, before sending them out. After the last data bit is sent, the stop bit is used to end the frame and the line returns to the idle state.

-   7-bit ASCII 'S' (0x52) = 1 0 1 0 0 1 1
-   LSB order = 1 1 0 0 1 0 1


![2024-07-14 12_43_29-Understanding UART _ Rohde   Schwarz - Brave](https://github.com/user-attachments/assets/97853c5d-5b8e-4719-b3b1-705122ec32d7)



#### Parity bit

A UART frame can also contain an optional parity bit that can be used for error detection. This bit is inserted between the end of the data bits and the stop bit. The value of the parity bit depends on the type of parity being used (even or odd):

-   In even parity, this bit is set such that the total number of 1s in the frame will be even.
-   In odd parity, this bit is set such that the total number of 1s in the frame will be odd.

Example:\
Capital "S" (1 0 1 0 0 1 1) contains a total of three zeros and 4 ones. If using even parity, the parity bit is zero because there already is an even number of ones. If using odd parity, then the parity bit has to be one in order to make the frame have an odd number of 1s.\
The parity bit can only detect a single flipped bit. If more than one bit is flipped, there's no way to reliably detect these using a single parity bit.


![2024-07-14 12_44_13-Understanding UART _ Rohde   Schwarz - Brave](https://github.com/user-attachments/assets/43140f6e-03f0-404d-894d-fecb83ecd83d)

### STOP BITS

To signal the end of the data packet, the sending UART drives the data transmission line from a low voltage to a high voltage for at least two bit durations.


STEPS OF UART TRANSMISSION
--------------------------

1\. The transmitting UART receives data in parallel from the data bus:

[![Introduction to UART - Data Transmission Diagram UART Gets Byte from Data Bus](https://www.circuitbasics.com/wp-content/uploads/2016/01/Introduction-to-UART-Data-Transmission-Diagram-UART-Gets-Byte-from-Data-Bus-300x291.png)](https://www.circuitbasics.com/wp-content/uploads/2016/01/Introduction-to-UART-Data-Transmission-Diagram-UART-Gets-Byte-from-Data-Bus.png)

2\. The transmitting UART adds the start bit, parity bit, and the stop bit(s) to the data frame:

[![Introduction to UART - Data Transmission Diagram UART Adds Start, Parity, ad Stop Bits](https://www.circuitbasics.com/wp-content/uploads/2016/01/Introduction-to-UART-Data-Transmission-Diagram-UART-Adds-Start-Parity-ad-Stop-Bits-2-300x179.png)](https://www.circuitbasics.com/wp-content/uploads/2016/01/Introduction-to-UART-Data-Transmission-Diagram-UART-Adds-Start-Parity-ad-Stop-Bits-2.png)

3\. The entire packet is sent serially from the transmitting UART to the receiving UART. The receiving UART samples the data line at the pre-configured baud rate:

[![Introduction to UART - Data Transmission Diagram Transmitting UART Sends Data Packet Serially to Receiving UART](https://www.circuitbasics.com/wp-content/uploads/2016/01/Introduction-to-UART-Data-Transmission-Diagram-Transmitting-UART-Sends-Data-Packet-Serially-to-Receiving-UART-300x139.png)](https://www.circuitbasics.com/wp-content/uploads/2016/01/Introduction-to-UART-Data-Transmission-Diagram-Transmitting-UART-Sends-Data-Packet-Serially-to-Receiving-UART.png)

4\.  The receiving UART discards the start bit, parity bit, and stop bit from the data frame:

[![Introduction to UART - Data Transmission Diagram UART Removes Start, Parity, and Stop Bits](https://www.circuitbasics.com/wp-content/uploads/2016/01/Introduction-to-UART-Data-Transmission-Diagram-UART-Removes-Start-Parity-and-Stop-Bits-2-300x179.png)](https://www.circuitbasics.com/wp-content/uploads/2016/01/Introduction-to-UART-Data-Transmission-Diagram-UART-Removes-Start-Parity-and-Stop-Bits-2.png)

5\. The receiving UART converts the serial data back into parallel and transfers it to the data bus on the receiving end:

[![Introduction to UART - Data Transmission Diagram Receiving UART Sends Byte to Data Bus](https://www.circuitbasics.com/wp-content/uploads/2016/02/Introduction-to-UART-Data-Transmission-Diagram-Receiving-UART-Sends-Byte-to-Data-Bus-2-286x300.png)](https://www.circuitbasics.com/wp-content/uploads/2016/02/Introduction-to-UART-Data-Transmission-Diagram-Receiving-UART-Sends-Byte-to-Data-Bus-2.png)

ADVANTAGES AND DISADVANTAGES OF UARTS
-------------------------------------

No communication protocol is perfect, but UARTs are pretty good at what they do. Here are some pros and cons to help you decide whether or not they fit the needs of your project:

### ADVANTAGES

-   Only uses two wires
-   No clock signal is necessary
-   Has a parity bit to allow for error checking
-   The structure of the data packet can be changed as long as both sides are set up for it
-   Well documented and widely used method

### DISADVANTAGES

-   The size of the data frame is limited to a maximum of 9 bits
-   Doesn't support multiple slave or multiple master systems
-   The baud rates of each UART must be within 10% of each other

### Basics of UART Communication
**UART Overview**:

-   UART is a hardware communication protocol that uses asynchronous serial communication with configurable speed.
-   It does not use a clock signal for synchronization. Instead, the transmitter generates a bitstream based on its clock signal, while the receiver uses its internal clock to sample the incoming data.
-   For effective communication, both transmitting and receiving devices must have the same baud rate, which is the rate at which information is transferred.

**Key UART Components**:

1.  **Transmitter (Tx)**: Sends data serially bit by bit.
2.  **Receiver (Rx)**: Receives serial data and converts it back to parallel form.

**Standard UART Configuration**:

-   Wires: 2 (Tx and Rx)
-   Speed: Common baud rates include 9600, 19200, 38400, 57600, 115200, 230400, 460800, 921600, 1000000, 1500000
-   Method of Transmission: Asynchronous
-   Maximum Number of Masters: 1
-   Maximum Number of Slaves: 1

### Data Transmission Process

**UART Packet Structure**:

-   **Start Bit**: Initiates the data transfer by pulling the line low.
-   **Data Frame**: Contains the actual data (5 to 9 bits long, depending on the use of a parity bit).
-   **Parity Bit**: Optional; used for error checking by ensuring the total number of 1s is even (even parity) or odd (odd parity).
-   **Stop Bits**: Signify the end of the packet by pulling the line high for 1 or 2 bit durations.

**Transmission Steps**:

1.  The transmitting UART receives parallel data from the data bus.
2.  It adds the start bit, parity bit, and stop bit(s) to the data frame.
3.  The packet is sent serially from the transmitting UART to the receiving UART.
4.  The receiving UART samples the data line at the baud rate, discards the start, parity, and stop bits.
5.  It converts the serial data back to parallel and transfers it to the data bus.

### Frame Protocol

A frame protocol adds security and error-checking features to the UART communication. It includes a header, command, data length, data, trailer, and CRC (Cyclic Redundancy Check). Each device can have unique configurations for these elements to ensure secure communication.

**Frame Protocol Components**:

-   **Header**: Unique identifier for the device.
-   **Command**: List of commands for communication between devices.
-   **Data Length**: Varies based on the command.
-   **Data**: Payload to be transferred.
-   **Trailer**: Indicates the end of the transmission.
-   **CRC**: Error-checking formula to ensure data integrity.

### UART Operations

**Steps for Implementing UART**:

1.  Check the device's datasheet for UART interface details.
2.  Locate the UART address in the memory map.
3.  Review the UART port details (operation mode, data bits length, parity bit, stop bits).
4.  Configure the baud rate using the formula provided in the datasheet.
5.  Ensure both devices use the same baud rate to avoid timing discrepancies.
6.  Check the detailed registers for UART configuration (e.g., UART_DIV, UART_FIBR, UART_LCR2).
7.  Substitute values to compute the baud rate and start implementing UART.

### Importance and Applications

**Importance**:

-   Understanding UART is crucial for developing reliable and efficient embedded systems.
-   Ensures robust data transfer and minimizes errors.

**Applications**:

-   **Debugging**: Capture system messages for early bug detection.
-   **Manufacturing Tracing**: Logs provide insights into manufacturing processes.
-   **Customer Updates**: Enables dynamic hardware with update-capable software.
-   **Testing/Verification**: Ensures product quality before delivery.
