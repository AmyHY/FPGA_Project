# FPGA_Project
ECE 385 FINAL PROJECT REPORT

## Introduction

For our final project, we aim to create a game using concepts taught in the labs of ECE 385. The game is run on the Spartan 7 board, and uses a SystemOnChip approach with SystemVerilog being used as the hardware and C code as software. This game is a simple target shooter. It consists of a spaceship which is controlled by the user, which can be used to move or shoot. The targets are meteors, which appear randomly from the top of the screen and are intended to damage the spaceship. This spaceship has to contend with these meteors launched at it which it must aim to shoot down for a score, and contact with this meteor would result in a loss of a life. The loss of three lives marks the end of the game and the player has the option to start a new one.

## Functionality Plans

As our game requires an interface to show the game on, we used Lab 6.2 as the base of our game as it contains the VGA to HDMI IP conversion, the Microblaze block diagram necessary to do so, and the MAX3421E functions that help connect our USB keyboard to the FPGA. Our game design was to be primarily done on hardware through SystemVerilog in Vivado. We used SystemVerilog for both graphics and game logic.

## VGA operation

The VGA operation involves the VGA controller, Color Mapper, and the Ball module. The VGA controller generates video signals based on resolution and coordinates, reading RGB values from the Color Mapper. It iterates through DrawX and DrawY to draw pixels, synchronizing with monitor sync signals. The Ball module provides BallX and BallY. The VGA controller uses DrawX and DrawY as inputs for the Color Mapper, which generates RGB values for each pixel. This interaction results in a coherent video stream displaying the ball on the screen, with the Color Mapper providing color data for the VGA controller to render the ball and background images accurately.

## VGA-HDMI IP

VGA-HDMI IP is an IP block which converts the VGA signal to an HDMI signal. They share similarities in terms of their interface, function of transmitting video signals, compatibility through converters, and the use of pins. HDMI is a digital interface whilst VGA is an analog one, so we can provide higher image quality.

## SV Module description

### Module: new_game.sv
- **Inputs:**  input logic Clk, Reset, logic has_collision, input logic [7:0] keycode,  
- **Outputs:**  output logic [2:0] game_state,  logic reset_game  
- **Description:**  This module defines the overall structure and rules of the game. Through the use of enum logic some states of the game, and the reset state is exited on pressing the enter key which starts the game. The state machine goes to the next state when a collision is detected. After the game is over it goes back to the reset state by default

### Module: spaceship_movement.sv  
- **Inputs:**  logic [7:0] keycode,  
- **Outputs:**  output logic [9:0]  SpaceshipX, SpaceshipY, SpaceshipS, output logic SpaceshipOn  
- **Description:**  This module defines logic for the spaceship. It sets parameters such that it is constrained to lie in the bottom of the screen and centered at Y = 432 pixels. Using the keycodes it can go left when “a'' is pressed and right when “d” is pressed. An additional functionality we did not implement was the shoot option which would be controlled by the “space” button.

### Module: meteor_movement.sv  
- **Inputs:**  logic Reset, frame_clk  
- **Outputs:**  output logic [9:0]  meteorX, meteorY  
- **Description:**  This code instantiates a single meteor that comes from the top. This code needed further changing to account for the randomisation of multiple meteors arriving from the top of the screen

### Module: collision.sv  
- **Inputs:**  input logic frame_clk, logic [9:0] SpaceshipX, SpaceshipY, MeteorX, MeteorY,  
- **Outputs:**  logic Collision  
- **Description:**  When the meteor and the spaceship pixels interlap it counts as a collision. We use the Spaceship and Bullets parameters which we defined for their previous modules to find these points and the conditions for overlap

### Module: new_color_mapper.sv  
- **Inputs:**  logic [9:0] DrawX, DrawY, SpaceshipX, SpaceshipY, SpaceshipS, BulletX, BulletY, logic SpaceshipOn, Shooting, Clk  
- **Outputs:**  logic [7:0]  Red, Green, Blue  
- **Description:**  This module assigns the color for the screen. It keeps a track of what pixel we are on through DrawX and DrawY and depending on whether these pixels lie within the radius of the ball, it will assign color in that pixel.It works much like ball.sv in Lab 6.2, and we check whether the bullet/spaceship or design is present and will assign colors accordingly. In this case the spaceship and meteor colors are read from spaceship_reader and meteor_reader.

### Module: spaceship_reader.sv  
- **Inputs:**  [18:0] read_address, input Clk,  
- **Outputs:**  logic [23:0] data_Out  
- **Description:**  Instantiates a reader that defines logic and access and reads in the hex values from our generated .mem files that show the spaceship on the screen

### Module: meteor_reader.sv  
- **Inputs:**  [18:0] read_address, input Clk,  
- **Outputs:**  logic [23:0] data_Out  
- **Description:**  Instantiates a reader that defines logic and access and reads in the hex values from our generated .mem files that show the meteor on the screen
MEM files: R,G,and B hex files used to display our image on screen. We used the [PNG - to Hex Converter]([URL](https://github.com/Atrifex/ECE385-HelperTools)) for this.  
- **Inputs:**  logic Reset, frame_clk,  input logic [7:0] keycode  
- **Outputs:**  logic [9:0]  BallX, BallY, BallS  
- **Description:**  This module first defines some parameters like the center of the screen, topmost, bottommost, leftmost, and rightmost edges of the screen and make it such that if the edge of the ball touches that the motion of the ball should stop. Then using the hex values of the w,s,d,s  which are up, down, right, and left directions respectively we define conditional logic such that depending on the keycode is pressed we choose that direction.

### Module: hex.sv  
- **Inputs:**  logic Clk,input logic, reset  
- **Outputs:**  [7:0]   hex_seg,   [3:0]   hex_grid  
- **Description:**  It uses a submodule nibble_to_hex to map a 4 bit nibble input to 7 segment display (for each hex digit). This in turn allows us to display hex values based on the input bits using a counter and case statements.

### Module: VGA_controller.sv  
- **Inputs:**  input pixel_clk,  reset  
- **Outputs:**  logic hs  
- **Description:**  The module tracks the screen position using vertical and horizontal counters and checks if it is okay to ‘display’ something on the screen there.

## Block Design Description

- **microblaze_0:** Deploys the MicroBlaze processor core with predefined settings.
- **mdm_1:** Instantiates the MicroBlaze Debug Module, streamlining debugging processes.
- **clk_wiz_1:** Initializes the Clocking Wizard to manage microcontroller clock signals.
- **microblaze_0_local_memory:** Manages the local memory module for the MicroBlaze core.
- **rst_clk_wiz_1_100M:** A foundational module configuring clocking and resets for the entire FPGA design.
- **timer_usb_axi:** Provides the MicroBlaze core with timekeeping capabilities in milliseconds, crucial for USB operations.
- **microblaze_0_axi_periph:** Represents the link between the MicroBlaze core and the AXI peripheral interconnect.
- **xlconcat_0:** Combines multiple signals or data sources into a wider data bus.
- **microblaze_0_axi_intc:** The MicroBlaze AXI Interrupt Controller, managing interrupts from peripherals and sources.
- **axi_uartlite_0:** Enables serial communication between the MicroBlaze processor and peripherals.
- **gpio_usb_rst:** AXI GPIO Input/Output module for USB initialization.
- **gpio_usb_int:** AXI GPIO Input/Output module for USB initialization.
- **gpio_usb_keycode:** A dual-channel GPIO module facilitating the transmission of up to 8 keycodes.
- **Spi_usb:** Manages the SPI peripheral for communication with the MAX3421E chip.

## Top Level Block Design  
Left Side:
![Left side top level block design](images/image3.jpg)

Right Side:
![Right side top level block design](images/image2.jpg)
![Right side top level block design_continued](images/image1.jpg)

## Design Resources and Statistics  
| Resource         | Usage                |
|------------------|----------------------|
| LUT              | 1407                 |
| DSP              | 15                   |
| Memory (BRAM)    | 0                    |
| Flip-Flops       | 103                  |
| Latches          | 1                    |
| Frequency        | 1/(10ns - 1.747ns) = 0.121 GHz |
| Static Power     | 0.075 W              |
| Dynamic Power    | 0.395 W              |
| Total Power      | 0.471 W              |

## Conclusion  
We successfully completed our game, overcoming challenges along the way. While our game logic was solid, integrating images onto the screen posed a significant hurdle. In hindsight, focusing on loading images into Block RAM (BRAM) as a foundational step would have been beneficial.

Ultimately, we opted to use a PNG to Hex Converter at the last minute to generate memory files, which allowed us to store image data locally instead of in BRAM. This decision enabled correct spaceship movement and display, along with the successful animation of a single meteor.

Moving forward, for more advanced games with enhanced graphics, leveraging BRAM for image storage and considering image compression techniques would be essential. This approach would facilitate smoother gameplay and more efficient use of resources.

