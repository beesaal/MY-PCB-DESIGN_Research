### Block Diagram Overview


![Board Overview with FPGA](https://github.com/user-attachments/assets/ca1b505d-1f3a-4fce-9604-96fcfde150c6)


[Board Overview with FPGA.pdf](https://github.com/user-attachments/files/16422740/Board.Overview.with.FPGA.pdf)




### Key Connections and Circuitry

1.  **ESP32 to FPGA (SPI Interface)**:

    -   **Purpose**: Transfer camera data and other commands via SPI.
    -   **Connections**:
        -   ESP32 SPI Master to FPGA SPI Slave.
2.  **FPGA to SDRAM (Parallel Interface)**:

    -   **Purpose**: Store camera data and provide buffering.
    -   **Connections**:
        -   Data, Address, and Control lines from FPGA to SDRAM.
3.  **FPGA to Camera Module**:

    -   **Purpose**: Receive image data from the camera module.
    -   **Connections**:
        -   Camera module interface (24-pin FFC) to FPGA.
4.  **FPGA to STM32 MCU (I2C/SPI/GPIO)**:

    -   **Purpose**: Coordinate data processing and control tasks.
    -   **Connections**:
        -   SPI or I2C for command and data transfer.
        -   GPIO for control signals.
5.  **STM32 MCU to LCD Display**:

    -   **Purpose**: Display processed data.
    -   **Connections**:
        -   SPI, I2C, or Parallel interface to LCD Display.

### Circuitry Details

1.  **Power Supply**:

    -   Ensure proper power supply for all components (3.3V or 1.8V for FPGA, as required).
    -   Use voltage regulators for stable power.
2.  **FPGA Pin Assignments**:

    -   Assign FPGA I/O pins for SPI, parallel data lines, and control signals.
    -   Configure FPGA internal logic using Verilog/VHDL.
3.  **SDRAM Interface**:

    -   Connect data lines (D0-D15), address lines (A0-A12), and control signals (CAS, RAS, WE) between FPGA and SDRAM.
    -   Implement timing control within FPGA.
4.  **Camera Module Interface**:

    -   Connect the camera module's data output lines to FPGA inputs.
    -   Use appropriate logic levels and ESD protection.
5.  **MCU Communication**:

    -   Connect SPI/I2C lines between STM32 MCU and FPGA.
    -   Use additional GPIOs for control signals (e.g., interrupts).

### Example Circuit Snippets

1.  **FPGA to SDRAM (Example Connections)**:

```sh

`FPGA         SDRAM
----         -----
D0-D15  <--> D0-D15
A0-A12  <--> A0-A12
WE       --> WE
CAS      --> CAS
RAS      --> RAS
CLK      --> CLK`
```

1.  **FPGA to Camera Module (Example Connections)**:

```sh

`FPGA         Camera Module
----         --------------
CAM_D0-D7 <--> Data Pins
CAM_CLK   --> Clock Pin
CAM_VSYNC --> VSync Pin
CAM_HSYNC --> HSync Pin`
```

1.  **FPGA to MCU (SPI Example Connections)**:

```sh

`FPGA         STM32 MCU
----         ---------
SPI_MOSI <--> SPI_MOSI
SPI_MISO <--> SPI_MISO
SPI_SCLK <--> SPI_SCLK
SPI_CS   -->  SPI_CS
GPIOs    <--> Control Pins`
```

### Summary

By connecting the LCMXO2-640HC-4SG48C FPGA with the ESP32, SDRAM, camera module, STM32 MCU, and LCD display using the described interfaces and connections, you can effectively handle tasks like SPI to parallel conversion, data buffering, memory control, and real-time data processing. The FPGA acts as a versatile intermediary, processing data and facilitating communication between various components.
