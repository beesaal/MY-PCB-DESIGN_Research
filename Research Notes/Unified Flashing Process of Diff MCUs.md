# Updated Process


![Flashing](https://github.com/user-attachments/assets/01d317de-fcad-43a9-a900-5c2a5ab6d83b)

The updated process involves the following key elements:

1.  **Modular Firmware Management**: Split the task into distinct phases: storing firmware, flashing devices, and verifying the flashing process.
2.  **Fault Tolerance**: Implement error checking and recovery mechanisms during flashing.
3.  **Scalability**: Make the system scalable for adding more devices or upgrading the firmware.

### Process

#### **1\. Initial Setup**

-   **External Flash Memory**: Store the firmware binaries for ESP32 and FPGA in specific memory sectors.
-   **Boot Control**: Designate GPIOs for controlling the boot modes of ESP32 and FPGA.
-   **LED Indicators**: Use different LEDs to indicate which chip is being flashed or if an error occurs.

#### **2\. Programming the STM32H7B0VB**

-   **Phase 1: Flash Storage**:

    -   Load the firmware for ESP32 and FPGA into the external flash memory.
    -   Store firmware addresses and metadata in a reserved memory sector for easy retrieval.
-   **Phase 2: Flashing the Devices**:

    -   **Flashing ESP32**:
        1.  Configure ESP32 boot pins for flashing mode.
        2.  Transfer the firmware from external flash to ESP32 via UART or SPI.
        3.  Validate the transfer with checksums or other verification methods.
        4.  Signal success or failure using LED indicators.
    -   **Flashing FPGA**:
        1.  Configure FPGA JTAG pins for configuration mode.
        2.  Transfer the bitstream from external flash to FPGA via JTAG.
        3.  Validate the configuration.
        4.  Signal success or failure using LED indicators.
-   **Phase 3: Operation Mode**:

    -   After flashing, the STM32H7B0VB resumes normal operation.
    -   Reset ESP32 and FPGA to start their main tasks.
    -   Use LEDs to indicate normal operation.

#### **3\. Fault Tolerance**

-   Implement error detection at each step (e.g., checksum verification after each firmware transfer).
-   Introduce a retry mechanism in case of failures.
-   Use a watchdog timer to ensure the system doesn't hang during the process.

### Code Implementation

#### **1\. External Flash Memory Management**

-   **Function to Write Firmware to External Flash**:

```c

void WriteFirmwareToExternalFlash(uint8_t *data, size_t size, uint32_t address) {
    // Chip select and SPI communication to write data
    HAL_GPIO_WritePin(FLASH_CS_PIN, GPIO_PIN_RESET);
    HAL_SPI_Transmit(&hspi2, data, size, HAL_MAX_DELAY);
    HAL_GPIO_WritePin(FLASH_CS_PIN, GPIO_PIN_SET);
}

void StoreFirmwareMetadata(uint32_t esp32_address, uint32_t esp32_size,
                           uint32_t fpga_address, uint32_t fpga_size) {
    FirmwareMetadata metadata = {
        .esp32_address = esp32_address,
        .esp32_size = esp32_size,
        .fpga_address = fpga_address,
        .fpga_size = fpga_size
    };
    WriteFirmwareToExternalFlash((uint8_t*)&metadata, sizeof(metadata), METADATA_FLASH_ADDRESS);
}
```

#### **2\. Flashing the ESP32**

```c



void ProgramESP32(void) {
    // Pull GPIO0 low and reset ESP32 to enter bootloader mode
    HAL_GPIO_WritePin(ESP32_BOOT_PIN, GPIO_PIN_RESET);
    HAL_GPIO_WritePin(ESP32_RESET_PIN, GPIO_PIN_RESET);
    HAL_Delay(100);
    HAL_GPIO_WritePin(ESP32_RESET_PIN, GPIO_PIN_SET);

    // Transfer firmware from external flash to ESP32 over UART
    uint32_t address = esp32_firmware_addr;
    uint8_t firmware_data[CHUNK_SIZE];
    for (size_t i = 0; i < esp32_firmware_size; i += CHUNK_SIZE) {
        ReadFromExternalFlash(firmware_data, CHUNK_SIZE, address);
        HAL_UART_Transmit(&huart3, firmware_data, CHUNK_SIZE, HAL_MAX_DELAY);
        address += CHUNK_SIZE;
    }

    // Verify the firmware
    if (VerifyESP32Firmware()) {
        HAL_GPIO_WritePin(ESP32_SUCCESS_LED, GPIO_PIN_SET);
    } else {
        HAL_GPIO_WritePin(ESP32_ERROR_LED, GPIO_PIN_SET);
    }

    // Release ESP32 from boot mode
    HAL_GPIO_WritePin(ESP32_BOOT_PIN, GPIO_PIN_SET);
}

bool VerifyESP32Firmware(void) {
    // Implement a checksum or other verification method
    // Return true if successful, false otherwise
    return true;
}
```

#### **3\. Flashing the FPGA**

```c


void ProgramFPGA(void) {
    // Configure FPGA for flashing
    HAL_GPIO_WritePin(FPGA_CONFIG_PIN, GPIO_PIN_SET);

    // Transfer bitstream
    uint32_t address = fpga_bitstream_addr;
    uint8_t bitstream_data[CHUNK_SIZE];
    for (size_t i = 0; i < fpga_bitstream_size; i += CHUNK_SIZE) {
        ReadFromExternalFlash(bitstream_data, CHUNK_SIZE, address);
        HAL_SPI_Transmit(&hspi2, bitstream_data, CHUNK_SIZE, HAL_MAX_DELAY);
        address += CHUNK_SIZE;
    }

    // Verify FPGA configuration
    if (VerifyFPGAConfiguration()) {
        HAL_GPIO_WritePin(FPGA_SUCCESS_LED, GPIO_PIN_SET);
    } else {
        HAL_GPIO_WritePin(FPGA_ERROR_LED, GPIO_PIN_SET);
    }

    // Release FPGA from configuration mode
    HAL_GPIO_WritePin(FPGA_CONFIG_PIN, GPIO_PIN_RESET);
}

bool VerifyFPGAConfiguration(void) {
    // Implement a verification method for FPGA bitstream
    // Return true if successful, false otherwise
    return true;
}
```

#### **4\. Operation Mode**

```c


void StartMainTask(void) {
    // Initialize and start the main application tasks for STM32
    InitializePeripherals();
    StartRTOS();

    // Reset ESP32 and FPGA to start their main tasks
    HAL_GPIO_WritePin(ESP32_RESET_PIN, GPIO_PIN_RESET);
    HAL_Delay(100);
    HAL_GPIO_WritePin(ESP32_RESET_PIN, GPIO_PIN_SET);

    HAL_GPIO_WritePin(FPGA_RESET_PIN, GPIO_PIN_RESET);
    HAL_Delay(100);
    HAL_GPIO_WritePin(FPGA_RESET_PIN, GPIO_PIN_SET);

    // Set the LED to indicate normal operation
    HAL_GPIO_WritePin(NORMAL_OPERATION_LED, GPIO_PIN_SET);
}

void InitializePeripherals(void) {
    // Initialize GPIOs, UART, SPI, and other peripherals
}

void StartRTOS(void) {
    // Start the RTOS scheduler
}
```

### Summary of the Process

1.  **STM32H7B0VB Initial Flashing**:

    -   Flash the STM32H7B0VB using ST-Link with a firmware that handles external flash memory management and the programming of other chips.
2.  **Storing Firmware in External Flash**:

    -   Store ESP32 and FPGA firmware in external flash memory.
    -   Store metadata about firmware location and size for easy retrieval.
3.  **Flashing ESP32**:

    -   Configure ESP32 for bootloader mode using GPIO pins.
    -   Transfer the firmware via UART or SPI.
    -   Verify the transfer and signal success or failure with LEDs.
4.  **Flashing FPGA**:

    -   Configure FPGA for JTAG mode.
    -   Transfer the bitstream via SPI/JTAG.
    -   Verify the configuration and signal success or failure with LEDs.
5.  **Post-Flashing Operation**:

    -   Reset all chips and start their main tasks.
    -   Use an LED to indicate normal operation.
6.  **Fault Tolerance**:

    -   Implement error detection and retry mechanisms.
    -   Use a watchdog timer to prevent system hangs.

This approach ensures that the flashing process is robust, easy to manage, and scalable for future development.

### **Complete Demo Code for STM32CubeIDE**

```c


#include "main.h"
#include "spi.h"
#include "usart.h"

// Define external flash memory layout
#define FLASH_BASE_ADDRESS      0x00000000
#define ESP32_FIRMWARE_ADDRESS  (FLASH_BASE_ADDRESS + 0x00000000)
#define FPGA_BITSTREAM_ADDRESS  (FLASH_BASE_ADDRESS + 0x00200000)
#define METADATA_FLASH_ADDRESS  (FLASH_BASE_ADDRESS + 0x7FFFF)     // Metadata at end

#define CHUNK_SIZE              256  // Adjust based on flash and SPI capabilities

// Example firmware and bitstream data (in practice, load these from an external source)
uint8_t esp32_firmware[] = { /* ... */ };  // ESP32 firmware binary data
uint8_t fpga_bitstream[] = { /* ... */ };  // FPGA bitstream binary data

// Function prototypes
void WriteFirmwareToExternalFlash(uint8_t *data, size_t size, uint32_t address);
void ReadFromExternalFlash(uint8_t *buffer, size_t size, uint32_t address);
void StoreFirmwareInFlash(void);
void ProgramESP32(void);
void ProgramFPGA(void);
void BlinkLED(uint8_t led_id, uint32_t duration);

// Main function
int main(void) {
    HAL_Init();
    SystemClock_Config();
    MX_GPIO_Init();
    MX_SPI2_Init();  // SPI for external flash and FPGA
    MX_USART3_UART_Init();  // UART for ESP32 flashing

    // Step 1: Store all firmware in external flash
    StoreFirmwareInFlash();

    // Step 2: Flash ESP32
    BlinkLED(1, 100);  // Indicate ESP32 flashing
    ProgramESP32();

    // Step 3: Flash FPGA
    BlinkLED(2, 100);  // Indicate FPGA flashing
    ProgramFPGA();

    // Step 4: Start STM32H7B0VB task applications
    // Add your application code here

    while (1) {
        // Main loop
    }
}

// Function to write firmware data to specific blocks of external flash
void WriteFirmwareToExternalFlash(uint8_t *data, size_t size, uint32_t address) {
    size_t chunk_size = CHUNK_SIZE;
    for (size_t offset = 0; offset < size; offset += chunk_size) {
        size_t write_size = (size - offset) > chunk_size ? chunk_size : (size - offset);
        HAL_GPIO_WritePin(FLASH_CS_GPIO_Port, FLASH_CS_Pin, GPIO_PIN_RESET);
        uint32_t current_address = address + offset;
        HAL_SPI_Transmit(&hspi2, (uint8_t*)&current_address, sizeof(current_address), HAL_MAX_DELAY);
        HAL_SPI_Transmit(&hspi2, data + offset, write_size, HAL_MAX_DELAY);
        HAL_GPIO_WritePin(FLASH_CS_GPIO_Port, FLASH_CS_Pin, GPIO_PIN_SET);
    }
}

// Function to read firmware data from specific blocks of external flash
void ReadFromExternalFlash(uint8_t *buffer, size_t size, uint32_t address) {
    HAL_GPIO_WritePin(FLASH_CS_GPIO_Port, FLASH_CS_Pin, GPIO_PIN_RESET);
    HAL_SPI_Transmit(&hspi2, (uint8_t*)&address, sizeof(address), HAL_MAX_DELAY);
    HAL_SPI_Receive(&hspi2, buffer, size, HAL_MAX_DELAY);
    HAL_GPIO_WritePin(FLASH_CS_GPIO_Port, FLASH_CS_Pin, GPIO_PIN_SET);
}

// Store ESP32 firmware and FPGA bitstream into external flash
void StoreFirmwareInFlash(void) {
    WriteFirmwareToExternalFlash(esp32_firmware, sizeof(esp32_firmware), ESP32_FIRMWARE_ADDRESS);
    WriteFirmwareToExternalFlash(fpga_bitstream, sizeof(fpga_bitstream), FPGA_BITSTREAM_ADDRESS);
    // Optionally store metadata for addresses and sizes
}

// Flash ESP32 using data from external flash
void ProgramESP32(void) {
    uint8_t firmware_data[CHUNK_SIZE];
    uint32_t address = ESP32_FIRMWARE_ADDRESS;
    for (size_t i = 0; i < sizeof(esp32_firmware); i += CHUNK_SIZE) {
        ReadFromExternalFlash(firmware_data, CHUNK_SIZE, address);
        HAL_UART_Transmit(&huart3, firmware_data, CHUNK_SIZE, HAL_MAX_DELAY);
        address += CHUNK_SIZE;
    }
    // Add verification or boot pin toggling if necessary
// Verify the firmware
    if (VerifyESP32Firmware()) {
        HAL_GPIO_WritePin(ESP32_SUCCESS_LED_GPIO_Port, ESP32_SUCCESS_LED_Pin, GPIO_PIN_SET);
    }
    else {
    HAL_GPIO_WritePin(ESP32_ERROR_LED_GPIO_Port, ESP32_ERROR_LED_Pin, GPIO_PIN_SET);
    }
    // Release ESP32 from boot mode
    HAL_GPIO_WritePin(ESP32_BOOT_GPIO_Port, ESP32_BOOT_Pin, GPIO_PIN_SET);
}
bool VerifyESP32Firmware(void) {
    // Implement a checksum or other verification method
    // Return true if successful, false otherwise return true;
}
    }

// Flash FPGA using data from external flash
void ProgramFPGA(void) {
    uint8_t bitstream_data[CHUNK_SIZE];
    uint32_t address = FPGA_BITSTREAM_ADDRESS;
    for (size_t i = 0; i < sizeof(fpga_bitstream); i += CHUNK_SIZE) {
        ReadFromExternalFlash(bitstream_data, CHUNK_SIZE, address);
        HAL_SPI_Transmit(&hspi2, bitstream_data, CHUNK_SIZE, HAL_MAX_DELAY);
        address += CHUNK_SIZE;
    }
    // Add verification or boot pin toggling if necessary
if (VerifyFPGAConfiguration()) {
        HAL_GPIO_WritePin(FPGA_SUCCESS_LED_GPIO_Port, FPGA_SUCCESS_LED_Pin, GPIO_PIN_SET);
    } else {
        HAL_GPIO_WritePin(FPGA_ERROR_LED_GPIO_Port, FPGA_ERROR_LED_Pin, GPIO_PIN_SET);
    }
    // Release FPGA from configuration mode
    HAL_GPIO_WritePin(FPGA_CONFIG_GPIO_Port, FPGA_CONFIG_Pin, GPIO_PIN_RESET);
}

bool VerifyFPGAConfiguration(void) {
    // Implement a verification method for FPGA bitstream
    // Return true if successful, false otherwise
    return true;

}


// Simple LED blink function to indicate flashing stages
void BlinkLED(uint8_t led_id, uint32_t duration) {
    // Implement LED blinking logic based on GPIO pin setup
    HAL_GPIO_WritePin(LED_GPIO_Port, led_id, GPIO_PIN_SET);
    HAL_Delay(duration);
    HAL_GPIO_WritePin(LED_GPIO_Port, led_id, GPIO_PIN_RESET);
}
```

#### ** Main Task Operation**

```c


void StartMainTask(void) {
    // Initialize and start the main application tasks for STM32
    InitializePeripherals();
    StartRTOS();

    // Reset ESP32 and FPGA to start their main tasks
    HAL_GPIO_WritePin(ESP32_RESET_GPIO_Port, ESP32_RESET_Pin, GPIO_PIN_RESET);
    HAL_Delay(100);
    HAL_GPIO_WritePin(ESP32_RESET_GPIO_Port, ESP32_RESET_Pin, GPIO_PIN_SET);

    HAL_GPIO_WritePin(FPGA_RESET_GPIO_Port, FPGA_RESET_Pin, GPIO_PIN_RESET);
    HAL_Delay(100);
    HAL_GPIO_WritePin(FPGA_RESET_GPIO_Port, FPGA_RESET_Pin, GPIO_PIN_SET);

    // Set the LED to indicate normal operation
    HAL_GPIO_WritePin(NORMAL_OPERATION_LED_GPIO_Port, NORMAL_OPERATION_LED_Pin, GPIO_PIN_SET);
}

void InitializePeripherals(void) {
    // Initialize GPIOs, UART, SPI, and other peripherals
}

void StartRTOS(void) {
    // Start the RTOS scheduler
    osKernelStart();
}
```

#### ** Error Handling**

```c

void Error_Handler(void) {
    // Toggle an error LED or send debug information via UART
    HAL_GPIO_WritePin(ERROR_LED_GPIO_Port, ERROR_LED_Pin, GPIO_PIN_SET);
    while(1) {
        HAL_Delay(500);
        HAL_GPIO_TogglePin(ERROR_LED_GPIO_Port, ERROR_LED_Pin);
    }
}
```

### **Summary**

1.  **Memory Layout:** The code specifies exact memory locations for storing the ESP32 firmware and FPGA bitstream in the external flash.

2.  **Writing to Flash:** The `WriteFirmwareToExternalFlash()` function handles the firmware storage in external flash, splitting data into manageable chunks if necessary.

3.  **Reading and Programming:** The `ProgramESP32()` and `ProgramFPGA()` functions read the firmware from the external flash memory and flash the ESP32 and FPGA, respectively.

4.  **LED Indication:** `BlinkLED()` is used to blink LEDs to indicate which chip is currently being flashed.

5.  **Main Function:** The `main()` function coordinates the storage of firmware, flashing of each chip, and eventually, the startup of the STM32H7B0VB's tasks.
