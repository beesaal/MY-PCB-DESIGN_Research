# *Unified Programming Architecture*


#### A. **Primary Interface**

-   **USB**: Use the USB interface of the STM32F103 (configured as ST-Link) as the primary connection between your PC and the programming hardware on the PCB.

#### B. **Multiplexer Control**

-   **Multiplexer (MUX)**: Use a multiplexer to switch the programming lines to the appropriate device.
-   **Control Signals**: Control the multiplexer selection lines using GPIO pins from the STM32F103.

### 2\. **Hardware Design**

#### A. **Connections**

1.  **STM32H7B0VB**

    -   **SWD**: SWDIO, SWCLK, GND, VCC
2.  **ESP32-S3-WROOM-1**

    -   **UART**: TX, RX, EN, IO0, GND, VCC
3.  **FPGA (LCMXO2-1200HC-4TG144C)**

    -   **JTAG**: TDI, TDO, TCK, TMS, GND, VCC
4.  **ST-Link MCU (STM32F103)**

    -   **USB**: Direct connection to PC for programming interface
    -   **GPIOs**: To control the MUX

#### B. **Multiplexer Selection**

-   **MUX IC**: Use a multiplexer like 74HC4066 to switch between programming lines.
-   **Control Lines**: Use GPIOs from STM32F103 to control the multiplexer.

### Example Block Diagram:

```sh

 `+-------------------+
        |   USB Interface   |
        |   (STM32F103)     |
        +--------+----------+
                 |
        +--------v----------+
        |    ST-Link MCU    |
        |    (STM32F103)    |
        +--------+----------+
                 | JTAG/SWD/UART
                 |
         +-------v--------+
         |  Multiplexer   |
         |  (74HC4066)    |
         +---+---+---+----+
             |   |   |
             |   |   |
             |   |   |
      +------+   |   +-------+
      |          |           |
      |          |           |
+-----v--+  +----v--+   +----v----+
|  STM32 |  |  ESP32 |   |  FPGA  |
| H7B0VB |  | S3-WROOM-1|   | LCMXO2|
| (SWD)  |  |  (UART)  |   | (JTAG) |
+--------+  +---------+   +---------+
      |
      | Control Signals
      | (from STM32F103)
      +----------------------------+`
```

### 3\. **Control Logic**

#### A. **STM32F103 GPIO Control**

-   **GPIOs**: Use GPIOs from STM32F103 to select the target device through the multiplexer.
-   **Selection Code**:

```sh

`// GPIO pins connected to the multiplexer control lines
#define MUX_SEL0_PIN GPIO_PIN_X
#define MUX_SEL1_PIN GPIO_PIN_Y

void select_device(int device) {
    switch (device) {
        case 0:
            // Select STM32H7B0VB
            HAL_GPIO_WritePin(GPIO_PORT, MUX_SEL0_PIN, GPIO_PIN_RESET);
            HAL_GPIO_WritePin(GPIO_PORT, MUX_SEL1_PIN, GPIO_PIN_RESET);
            break;
        case 1:
            // Select ESP32
            HAL_GPIO_WritePin(GPIO_PORT, MUX_SEL0_PIN, GPIO_PIN_SET);
            HAL_GPIO_WritePin(GPIO_PORT, MUX_SEL1_PIN, GPIO_PIN_RESET);
            break;
        case 2:
            // Select FPGA
            HAL_GPIO_WritePin(GPIO_PORT, MUX_SEL0_PIN, GPIO_PIN_RESET);
            HAL_GPIO_WritePin(GPIO_PORT, MUX_SEL1_PIN, GPIO_PIN_SET);
            break;
        default:
            break;
    }
}`
```
### 4\. **Programming Sequence**

1.  **Connect USB to PC**: Connect the USB interface of STM32F103 to the PC.
2.  **Control MUX**: Use STM32F103 to select the device via MUX.
3.  **Load Firmware**: Use appropriate software tools (like ST-Link Utility) to load firmware to the selected device.
4.  **Repeat**: Repeat the process for each device.

### 5\. **Using External Flash Memory**

You can partition the external flash memory to store firmware for each device and have them read their respective sections at boot. This requires a custom bootloader for each device to read the appropriate memory segment.

### Summary

By using the STM32F103 as an ST-Link MCU, you can program multiple devices on your PCB using a single USB interface. The process involves switching the programming lines using a multiplexer controlled by GPIOs from the STM32F103. This method streamlines the programming process and can be effectively implemented with careful design and control logic.
can partition the external flash memory to store firmware for each device and have them read their respective sections at boot. This requires a custom bootloader for each device to read the appropriate memory segment.
