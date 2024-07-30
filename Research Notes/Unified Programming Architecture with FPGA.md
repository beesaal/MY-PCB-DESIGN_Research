# *Unified Programming Architecture*


### Block Diagram:

![FPGA](https://github.com/user-attachments/assets/ba071461-b29c-4c3b-bd7e-b5d88b249cd4)

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

-   **MUX IC**: Use a multiplexer like SN74HC4066PWR to switch between programming lines.
-   **Control Lines**: Use GPIOs from STM32F103 to control the multiplexer

### 3\. **Control Logic**

#### A. **STM32F103 GPIO Control**

-   **GPIOs**: Use GPIOs from STM32F103 to select the target device through the multiplexer.
-   **Selection Code**:

```c

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

### Summary

By using the STM32F103 as an ST-Link MCU, you can program multiple devices on your PCB using a single USB interface. The process involves switching the programming lines using a multiplexer controlled by GPIOs from the STM32F103. This method streamlines the programming process and can be effectively implemented with careful design and control logic.
can partition the external flash memory to store firmware for each device and have them read their respective sections at boot. This requires a custom bootloader for each device to read the appropriate memory segment.



## Using External Flash Memory

You can partition the external flash memory to store firmware for each device and have them read their respective sections at boot. This requires a custom bootloader for each device to read the appropriate memory segment.

### *Routing and Connections*


1.  **SPI Interface to NOR Flash:**

    -   Connect MOSI, MISO, SCK, and CS pins from STM32F103 to W25Q128JVSIM.
    -   Ensure proper grounding and decoupling capacitors near the flash.
2.  **Programming Interfaces:**

    -   **STM32H7B0VB**: SWD pins (SWCLK, SWDIO) connected to STM32F103.
    -   **ESP32-S3-WROOM-1**: UART pins (TX, RX) connected to STM32F103.
    -   **FPGA**: JTAG pins (TCK, TDI, TDO, TMS) connected to STM32F103.
3.  **Multiplexer Control:**

    -   Use GPIOs from STM32F103 to control the multiplexer. Connect control lines to the multiplexer.
4.  **Boot Pins:**

    -   Design GPIO-controlled circuits to toggle boot pins for each device.

### *Power Supply and Decoupling*

-   Ensure stable power supply with proper decoupling capacitors for each IC.
-   Provide separate voltage rails if necessary for different devices.

##  *Programming Strategy*


#### **2.1. Firmware Storage**

1.  **Memory Block Allocation:**
    -   **STM32H7B0VB Firmware**: 0x00000000 - 0x000FFFFF
    -   **ESP32-S3-WROOM-1 Firmware**: 0x00100000 - 0x001FFFFF
    -   **FPGA Bitstream**: 0x00200000 - 0x002FFFFF

#### **2.2. Programming Flow**

1.  **Connect STM32F103 to PC**:

    -   Use USB to connect STM32F103 to a PC for programming and firmware management.
2.  **Control MUX and Boot Pins**:

    -   Use GPIOs from STM32F103 to select the target device through the multiplexer.
    -   Set boot pins as necessary to allow the target device to enter programming mode.
3.  **Read Firmware from NOR Flash**:

    -   Read the firmware image from the W25Q128JVSIM at the specific address.
    -   Transfer the firmware to the selected target device.
4.  **Write Firmware to Target Device**:

    -   Use SWD for STM32H7B0VB, UART for ESP32-S3-WROOM-1, and JTAG for FPGA.

### 3\. **Bootloader Design**

#### **3.1. Bootloader Overview**

-   **Purpose**: Allows updating of firmware for STM32H7B0VB, ESP32-S3-WROOM-1, and FPGA from the external NOR flash.
-   **Location**: Bootloader code resides in the STM32F103 and handles the programming of other chips.

#### **3.2. Bootloader Code Example**

**STM32F103 Bootloader:**

```c
            `#include "stm32f1xx_hal.h"
            
            // SPI handle
            extern SPI_HandleTypeDef hspi;
            
            // GPIO handle for controlling multiplexer
            extern GPIO_HandleTypeDef hgpio;
            
            // Addresses for firmware in NOR flash
            #define STM32H7_ADDRESS 0x00000000
            #define ESP32_ADDRESS 0x00100000
            #define FPGA_ADDRESS 0x00200000
            
            void select_device(int device) {
                // Set GPIOs for multiplexer control
                // Set boot pins as needed
            }
            
            void read_firmware(uint32_t address, uint8_t* buffer, uint32_t size) {
                // Read firmware from NOR flash
            }
            
            void program_device(uint32_t device) {
                uint32_t flash_address;
                uint32_t firmware_size;
                uint8_t buffer[256];  // Adjust buffer size as needed
            
                switch (device) {
                    case 0:
                        flash_address = STM32H7_ADDRESS;
                        firmware_size = STM32H7_FIRMWARE_SIZE;
                        break;
                    case 1:
                        flash_address = ESP32_ADDRESS;
                        firmware_size = ESP32_FIRMWARE_SIZE;
                        break;
                    case 2:
                        flash_address = FPGA_ADDRESS;
                        firmware_size = FPGA_BITSTREAM_SIZE;
                        break;
                    default:
                        return;
                }
            
                select_device(device);
            
                for (uint32_t i = 0; i < firmware_size; i += sizeof(buffer)) {
                    read_firmware(flash_address + i, buffer, sizeof(buffer));
                    // Write firmware to device
                    // Implement specific write functions for each device
                }
            }
            
            int main(void) {
                HAL_Init();
                // Initialize peripherals, SPI, GPIO, etc.
            
                while (1) {
                    // Wait for command to program a specific device
                    uint32_t device_to_program = get_device_to_program();
                    program_device(device_to_program);
                }
            }`

```
**Example Firmware Writing Functions:**

-   **STM32H7B0VB (SWD):**

    -   Use STM32CubeProgrammer API or write your own functions for SWD programming.
-   **ESP32-S3-WROOM-1 (UART):**

    -   Use ESP-IDF or a custom UART-based bootloader to write firmware.
-   **FPGA (JTAG):**

    -   Use Lattice Diamond tools or write custom JTAG programming functions.

#### **3.3. Bootloader Implementation**

**Bootloader for STM32H7B0VB:**

```c

            `void jump_to_application(uint32_t app_address) {
                // Prepare for application jump
                uint32_t jump_address = *(volatile uint32_t*)(app_address + 4);
                void (*jump_to_app)(void) = (void (*)(void))jump_address;
            
                // Set the main stack pointer
                __set_MSP(*(volatile uint32_t*)app_address);
                jump_to_app();
            }
            
            void bootloader_main(void) {
                if (*(volatile uint32_t*)STM32H7_ADDRESS != 0xFFFFFFFF) {
                    jump_to_application(STM32H7_ADDRESS);
                } else {
                    // Handle firmware update or wait for new firmware
                    while (1) {
                        // Wait for new firmware or user commands
                    }
                }
            }`
```
**Bootloader for ESP32:**

-   Customize the existing ESP32 bootloader or use a UART-based bootloader if necessary.

**Bootloader for FPGA:**

-   Use default Lattice tools or write custom logic for FPGA bitstream loading.

### Summary

By using a single external NOR flash memory for storing firmware and the STM32F103 as a central controller for programming different chips, you streamline your PCB design and programming process. Implementing a bootloader allows flexibility in firmware updates and provides a robust solution for managing multiple devices. This approach minimizes hardware complexity and leverages existing components efficiently.
