# Understanding FPGAs: A Deep Dive



Understanding how a programmed FPGA (Field-Programmable Gate Array) works involves diving into how its internal components, such as Configurable Logic Blocks (CLBs), interconnects, and I/O pins, interact to perform a task. Let's break down the process using a real-time example of image processing.

### FPGA Architecture Overview

1.  **Configurable Logic Blocks (CLBs)**: These are the basic building blocks of an FPGA. Each CLB contains look-up tables (LUTs), flip-flops, and multiplexers, which can be configured to perform various logic functions.
2.  **Interconnects**: These are the programmable wiring channels that connect CLBs to each other and to I/O pins.
3.  **I/O Pins**: These are the external connections that allow the FPGA to interface with other devices.

### Real-Time Example: Image Processing Task

#### Task: Simple Image Thresholding

**Objective**: Convert an image to black and white based on a threshold value. Each pixel's intensity is compared to a threshold. If the intensity is above the threshold, the pixel is set to white; otherwise, it's set to black.

#### Step-by-Step Breakdown

1.  **Input Data**: The image data is received via an SPI interface.
2.  **Processing Data**: The data is processed by comparing each pixel's intensity with a threshold value.
3.  **Output Data**: The processed image is output through a parallel interface.

#### Step 1: Receiving Image Data via SPI

Let's say we receive image data serially through the SPI interface. The SPI signals (MOSI, MISO, SCLK, and CS) are connected to specific I/O pins on the FPGA.

```verilog

module spi_receiver (
    input wire clk,
    input wire spi_mosi,
    input wire spi_clk,
    input wire spi_cs,
    output reg [7:0] pixel_data,
    output reg pixel_valid
);

// Internal registers for SPI
reg [7:0] shift_reg;
reg [2:0] bit_count;

always @(posedge spi_clk or posedge spi_cs) begin
    if (spi_cs) begin
        bit_count <= 3'b000;
        pixel_valid <= 1'b0;
    end else begin
        shift_reg <= {shift_reg[6:0], spi_mosi};
        bit_count <= bit_count + 1;
        if (bit_count == 3'b111) begin
            pixel_data <= shift_reg;
            pixel_valid <= 1'b1;
        end else begin
            pixel_valid <= 1'b0;
        end
    end
end
endmodule
```
Here, the SPI receiver captures serial data and assembles it into an 8-bit `pixel_data`. When a full byte is received, `pixel_valid` is asserted.

#### Step 2: Processing Image Data

Once the pixel data is received, we need to process it. Let's use a simple thresholding operation.

```verilog

module image_processor (
    input wire clk,
    input wire [7:0] pixel_data,
    input wire pixel_valid,
    output reg [7:0] processed_pixel
);

parameter THRESHOLD = 8'd128;

always @(posedge clk) begin
    if (pixel_valid) begin
        if (pixel_data > THRESHOLD) begin
            processed_pixel <= 8'hFF; // White
        end else begin
            processed_pixel <= 8'h00; // Black
        end
    end
end
endmodule

```

In this module, we compare `pixel_data` to a threshold value. If the data is above the threshold, we set `processed_pixel` to white (`0xFF`), otherwise to black (`0x00`).

#### Step 3: Outputting Processed Data

The processed pixel data is then sent out through a parallel interface.

```verilog

module parallel_output (
    input wire clk,
    input wire [7:0] processed_pixel,
    input wire pixel_valid,
    output reg [7:0] parallel_data,
    output reg data_ready
);

always @(posedge clk) begin
    if (pixel_valid) begin
        parallel_data <= processed_pixel;
        data_ready <= 1'b1;
    end else begin
        data_ready <= 1'b0;
    end
end
endmodule
```

This module outputs the processed pixel data on `parallel_data` and signals `data_ready` when valid data is available.

### Internal Processing in CLBs

To understand how CLBs handle this:

1.  **Input Pins to CLBs**: The SPI signals are routed to the CLBs via the FPGA's programmable interconnects.
2.  **Logic Implementation**: Inside the CLBs, LUTs are configured to implement the logic for the SPI receiver, threshold comparison, and parallel output.
3.  **Output Pins from CLBs**: The processed data is routed from the CLBs to the output pins.

#### Example: Threshold Comparison in CLB

-   **LUT Configuration**: The LUTs in the CLBs are programmed to perform the comparison operation (`pixel_data > THRESHOLD`).
-   **Flip-Flops**: The result of the comparison is stored in flip-flops and used to drive the output.

### Summary

-   **Data Flow**: SPI data is received, converted to parallel format, processed by comparing with a threshold, and output in parallel.
-   **CLBs**: Implement the logic for receiving, processing, and outputting data through configured LUTs and flip-flops.
-   **I/O Pins**: Facilitate communication with external devices for input and output data.

This example illustrates a basic FPGA image processing task. Real-world applications may involve more complex processing algorithms, multiple stages of data handling, and sophisticated interfacing techniques.

## *Other Examples*


### 1\. Binary to 7-Segment Display

**Objective**: Convert a 4-bit binary number into the corresponding 7-segment display code.

#### 1\. Define the Inputs and Outputs

We'll have a 4-bit binary input (`binary_input`) and a 7-bit output (`seg_output`) for the 7-segment display.

#### 2\. FPGA Top-Level Module

Let's define the top-level module that includes the input and output ports.

```verilog

module binary_to_7segment (
    input wire [3:0] binary_input,
    output reg [6:0] seg_output
);

    always @(*) begin
        case (binary_input)
            4'b0000: seg_output = 7'b0111111; // 0
            4'b0001: seg_output = 7'b0000110; // 1
            4'b0010: seg_output = 7'b1011011; // 2
            4'b0011: seg_output = 7'b1001111; // 3
            4'b0100: seg_output = 7'b1100110; // 4
            4'b0101: seg_output = 7'b1101101; // 5
            4'b0110: seg_output = 7'b1111101; // 6
            4'b0111: seg_output = 7'b0000111; // 7
            4'b1000: seg_output = 7'b1111111; // 8
            4'b1009: seg_output = 7'b1101111; // 9
            default: seg_output = 7'b0000000; // Blank
        endcase
    end

endmodule
  ```

### Explanation

1.  **Inputs and Outputs**:

    -   `binary_input`: 4-bit input representing a binary number.
    -   `seg_output`: 7-bit output to drive the segments of the 7-segment display.
2.  **Processing**:

    -   The `always` block executes whenever there is a change in `binary_input`.
    -   The `case` statement maps each 4-bit binary number to the corresponding 7-segment display code.
    -   The 7-segment display codes are predefined values where each bit represents a segment (`a-g`). A '1' means the segment is on, and a '0' means the segment is off.

### How It Works Inside the FPGA

#### Input Pins to CLBs

-   The 4-bit `binary_input` is connected to the FPGA's input pins. These pins route the signals to the CLBs.
-   Each bit of `binary_input` is connected to the LUTs inside the CLBs.

#### LUTs and Flip-Flops in CLBs

-   The LUTs in the CLBs are configured to implement the logic defined in the `case` statement.
-   For each possible value of `binary_input`, the corresponding output value (`seg_output`) is determined.
-   The flip-flops inside the CLBs hold the state of the `seg_output` until the next change in `binary_input`.

#### Output Pins from CLBs

-   The 7-bit `seg_output` is routed from the CLBs to the FPGA's output pins.
-   These pins drive the segments of the 7-segment display.

### Example of Data Flow

Let's walk through a specific example where `binary_input = 4'b0101` (which is the number 5):

1.  **Input**: The 4-bit value `0101` is applied to the input pins.
2.  **Processing**:
    -   The value `0101` is routed to the LUTs in the CLBs.
    -   The LUTs are configured to recognize `0101` and output the corresponding 7-segment code `1101101`.
3.  **Output**: The 7-bit value `1101101` is routed to the output pins, which drive the segments of the 7-segment display to show the number 5.

This simple example illustrates the core principles of how an FPGA processes data. By defining inputs, processing logic, and outputs, you can configure the FPGA to perform a wide range of tasks.




### 2\. **VGA Signal Generator**

**Objective**: Generate a simple VGA signal to display a pattern or text on a monitor.

**Explanation**: This project involves generating the required horizontal and vertical sync signals along with RGB color data to drive a VGA monitor.

**Verilog Code**:

```verilog

module vga_signal_generator(
    input wire clk,           // 25 MHz clock
    output wire h_sync,       // Horizontal sync signal
    output wire v_sync,       // Vertical sync signal
    output wire [2:0] rgb     // RGB color output
);

    // VGA 640x480 @ 60 Hz parameters
    parameter h_visible_area = 640;
    parameter h_front_porch = 16;
    parameter h_sync_pulse = 96;
    parameter h_back_porch = 48;
    parameter h_total = h_visible_area + h_front_porch + h_sync_pulse + h_back_porch;

    parameter v_visible_area = 480;
    parameter v_front_porch = 10;
    parameter v_sync_pulse = 2;
    parameter v_back_porch = 33;
    parameter v_total = v_visible_area + v_front_porch + v_sync_pulse + v_back_porch;

    reg [9:0] h_counter = 0;
    reg [9:0] v_counter = 0;

    always @(posedge clk) begin
        if (h_counter < h_total - 1)
            h_counter <= h_counter + 1;
        else begin
            h_counter <= 0;
            if (v_counter < v_total - 1)
                v_counter <= v_counter + 1;
            else
                v_counter <= 0;
        end
    end

    assign h_sync = (h_counter < (h_visible_area + h_front_porch)) ||
                    (h_counter >= (h_visible_area + h_front_porch + h_sync_pulse)) ? 1 : 0;

    assign v_sync = (v_counter < (v_visible_area + v_front_porch)) ||
                    (v_counter >= (v_visible_area + v_front_porch + v_sync_pulse)) ? 1 : 0;

    assign rgb = (h_counter < h_visible_area && v_counter < v_visible_area) ? 3'b111 : 3'b000;
endmodule
```



### 3\. **Digital Clock**

**Objective**: Create a digital clock that displays hours, minutes, and seconds on a 7-segment display.

**Explanation**: This project involves using a counter to keep track of time and displaying the values on 7-segment displays.

**Verilog Code**:

```verilog

module digital_clock(
    input wire clk,             // 1 Hz clock
    output wire [6:0] seg_hour1, // Hour tens place
    output wire [6:0] seg_hour0, // Hour ones place
    output wire [6:0] seg_min1,  // Minute tens place
    output wire [6:0] seg_min0,  // Minute ones place
    output wire [6:0] seg_sec1,  // Second tens place
    output wire [6:0] seg_sec0   // Second ones place
);

    reg [3:0] sec0 = 0, sec1 = 0, min0 = 0, min1 = 0, hour0 = 0, hour1 = 0;

    always @(posedge clk) begin
        if (sec0 == 9) begin
            sec0 <= 0;
            if (sec1 == 5) begin
                sec1 <= 0;
                if (min0 == 9) begin
                    min0 <= 0;
                    if (min1 == 5) begin
                        min1 <= 0;
                        if (hour0 == 9) begin
                            hour0 <= 0;
                            hour1 <= hour1 + 1;
                        end else if (hour0 == 3 && hour1 == 2) begin
                            hour0 <= 0;
                            hour1 <= 0;
                        end else begin
                            hour0 <= hour0 + 1;
                        end
                    end else begin
                        min1 <= min1 + 1;
                    end
                end else begin
                    min0 <= min0 + 1;
                end
            end else begin
                sec1 <= sec1 + 1;
            end
        end else begin
            sec0 <= sec0 + 1;
        end
    end

    // BCD to 7-segment decoder instance for each digit
    bcd_to_7seg hour1(.bcd(hour1), .seg(seg_hour1));
    bcd_to_7seg hour0(.bcd(hour0), .seg(seg_hour0));
    bcd_to_7seg min1(.bcd(min1), .seg(seg_min1));
    bcd_to_7seg min0(.bcd(min0), .seg(seg_min0));
    bcd_to_7seg sec1(.bcd(sec1), .seg(seg_sec1));
    bcd_to_7seg sec0(.bcd(sec0), .seg(seg_sec0));

endmodule

module bcd_to_7seg(
    input wire [3:0] bcd,
    output reg [6:0] seg
);
    always @(*) begin
        case (bcd)
            4'b0000: seg = 7'b0111111; // 0
            4'b0001: seg = 7'b0000110; // 1
            4'b0010: seg = 7'b1011011; // 2
            4'b0011: seg = 7'b1001111; // 3
            4'b0100: seg = 7'b1100110; // 4
            4'b0105: seg = 7'b1101101; // 5
            4'b0110: seg = 7'b1111101; // 6
            4'b0111: seg = 7'b0000111; // 7
            4'b1000: seg = 7'b1111111; // 8
            4'b1001: seg = 7'b1101111; // 9
            default: seg = 7'b0000000; // Blank
        endcase
    end
endmodule
```

### 4\. **PWM Generator for LED Dimming**

**Objective**: Create a Pulse Width Modulation (PWM) signal to control the brightness of an LED.

**Explanation**: This project generates a PWM signal with a variable duty cycle to adjust the brightness of an LED.

**Verilog Code**:

```verilog

module pwm_generator (
    input wire clk,            // System clock
    input wire [7:0] duty,     // Duty cycle (0-255)
    output reg pwm_out         // PWM output
);

    reg [7:0] counter = 0;

    always @(posedge clk) begin
        counter <= counter + 1;
        pwm_out <= (counter < duty) ? 1'b1 : 1'b0;
    end
endmodule
```

### Explanation of Each Project

#### VGA Signal Generator

-   **Inputs**: 25 MHz clock
-   **Outputs**: `h_sync`, `v_sync`, `rgb`
-   **Function**: Generates VGA timing signals and outputs a simple pattern.

#### Digital Clock

-   **Inputs**: 1 Hz clock
-   **Outputs**: `seg_hour1`, `seg_hour0`, `seg_min1`, `seg_min0`, `seg_sec1`, `seg_sec0`
-   **Function**: Keeps track of time and displays it on 7-segment displays using BCD (Binary-Coded Decimal).

#### PWM Generator

-   **Inputs**: System clock, 8-bit duty cycle
-   **Outputs**: `pwm_out`
-   **Function**: Generates a PWM signal to control the brightness of an LED.

These examples show how an FPGA can be used to perform a variety of tasks, from generating display signals to creating PWM signals for LED control. The key is to understand how to map your logic to the FPGA's resources, such as CLBs and I/O pins, to achieve the desired functionality.






# *Steps to Program an FPGA Using NOR Flash Memory*



1.  **Design and Synthesis**:

    -   Create your FPGA design using an HDL (like Verilog or VHDL).
    -   Synthesize the design to generate the bitstream file (binary file).
2.  **Store Bitstream in NOR Flash**:

    -   Store the generated bitstream file in a NOR flash memory. This can be done using a flash programmer or through an MCU.
3.  **Configure FPGA to Read from NOR Flash**:

    -   Set the FPGA configuration mode to read from external flash memory. This usually involves setting specific pins or configuration bits on the FPGA.
4.  **Power-Up Configuration**:

    -   When the FPGA powers up, it reads the bitstream from the NOR flash memory and configures itself according to the stored design.

### Example Workflow

#### 1\. Generate Bitstream

Assume you have a simple HDL design for an AND gate:

```verilog

module simple_and_gate(
    input wire a,
    input wire b,
    output wire c
);
    assign c = a & b;
endmodule
```

-   Synthesize this design using FPGA design tools (e.g., Xilinx Vivado, Intel Quartus).

#### 2\. Store Bitstream in NOR Flash

-   Use an FPGA tool to generate the bitstream file (e.g., `simple_and_gate.bit`).
-   Use a flash programmer or an MCU to write this bitstream file into the NOR flash memory connected to the FPGA.

#### 3\. Configure FPGA

-   Set the FPGA configuration mode to "Master SPI" or "Slave SPI" (or relevant mode for NOR flash).
-   This often involves setting specific M0, M1, and M2 pins on the FPGA.

### Example Code for MCU to Program NOR Flash

If using an MCU to write to NOR flash, you might have code like this:

```c

#include <spi.h> // Pseudo-code, specific libraries will depend on your MCU

#define FLASH_WRITE_ENABLE 0x06
#define FLASH_PAGE_PROGRAM 0x02
#define FLASH_SECTOR_ERASE 0x20
#define FLASH_READ_DATA 0x03

void write_enable_flash() {
    spi_send_command(FLASH_WRITE_ENABLE);
}

void erase_flash_sector(uint32_t address) {
    write_enable_flash();
    spi_send_command_with_address(FLASH_SECTOR_ERASE, address);
}

void program_flash_page(uint32_t address, uint8_t* data, size_t length) {
    write_enable_flash();
    spi_send_command_with_address_and_data(FLASH_PAGE_PROGRAM, address, data, length);
}

void write_bitstream_to_flash(uint8_t* bitstream, size_t length) {
    uint32_t address = 0x000000; // Start address
    size_t page_size = 256; // Example page size

    for (size_t i = 0; i < length; i += page_size) {
        erase_flash_sector(address + i);
        program_flash_page(address + i, bitstream + i, page_size);
    }
}

int main() {
    uint8_t bitstream[] = { /* Bitstream data */ };
    size_t length = sizeof(bitstream);

    write_bitstream_to_flash(bitstream, length);
    return 0;
}
```

### Conclusion

You can store the FPGA bitstream in NOR flash memory and configure the FPGA to read from it at power-up. This process is indeed called programming the FPGA. The bitstream tells the FPGA how to configure its internal logic and I/O pins to implement the desired functionality.
