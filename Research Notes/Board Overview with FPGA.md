# Board Overview with FPGA


### *Components List*
MCUs, SoC, FPGA:
```sh
        Main : STM32H7B0VBT6                (128KB 1.62V~3.6V 1.4MB 280MHz FLASH 80 LQFP-100(14x14))               $3.198
        SoC  : ESP32-S3-WROOM-1-N8          (ESP32-S3 Chip 2.4GHz SMD,18x25.5mm WiFi Modules ROHS)                 $3.825
        FPGA : LCMXO2-1200HC-4TG144C        (1280 160 TQFP-144(20x20) Programmable Logic Device (CPLDs/FPGAs))     $2.577
        MCU  : STM32F103C8T6                (64KB 2V~3.6V ARM-M3 20KB 72MHz FLASH 37 LQFP-48(7x7))                 $0.915


```
Memory:
```sh
        SDRAM          : W9825G6KH-6         (TSOP-54-10.2mm SDRAM ROHS)                                           $1.194
        Nor Flash ROM  : W25Q128JVSIM        (128Mbit SPI SOIC-8-208mil NOR FLASH ROHS)                            $0.8955
```
Expander/MUX
```sh
        IO Expander : MCP23S17-E/SO     (16 10MHz SPI SOIC-28-300mil I/O Expanders)                                $1.383 x2
        MUX         : SN74HC4066PWR     (30Ω 4 SPST(SPST)-Normal Open TSSOP-14 Analog Switches, Multiplexers)      $0.1308

```
Peripherals:
```sh
        Camera Module         : Ov2640
        MIC                   : ICS-43434         (Omni-directional 64dB - MEMS Microphones ROHS)                  $1.29
        IMU                   : QMA7981           (LGA-12(2x2) Accelerometers ROHS)                                $0.4935
        LCD                   : ILI9341 TFT
        HDMI-Interface        : ADV7513BSWZ       (LQFP-64(10x10) Video Interface ICs ROHS)                        $4.548
        HDMI-Port             : HDMI-01           (19P Female HDMI Horizontal attachment SMD D-Sub / VGA)          $0.2034
        GSM Module            : SIM900A
        SD-Card Slot          : TF PUSH           (2mm Card Slot MicroSD card (TF card) Self-bouncing, SMD)        $0.0443
        Ethernet-Transmitter  : DP83848CVVX/NOPB  (PHYtransceiver LQFP-48(7x7) Ethernet Transceivers ROHS)         $1.4835
        Ethernet-Port         : RC01931           (RJ45Receptacle 1 Shielded Direct Insert WithLED)                $0.2672
        RF-Module             : NRF24L01P-R       (2Mb/s SPI QFN- RF Transceiver ICs)                              $0.898
        USB Type-C Port       : USB4105-GF-A-060  (SMD USB Connectors ROHS)                                    frm toolroom

```


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
4.  **FPGA to STM32 MCU (FSMC/I2C/SPI/GPIO)**:

    -   **Purpose**: Coordinate data processing and control tasks.
    -   **Connections**:
        -   FSMC for random data access from FPGA.
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
