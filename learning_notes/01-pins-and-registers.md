# Board: STM32F411RE

## 1. Clock Enable
**The first important step**
On STM32, all peripherals (including GPIO ports) are disconnected fro the clock after reset to save power. If we try to awrite to a register without clock enable, the microcontroler ignores the wirte or an error occurs.
  
Where it is located: In the RCC (Reset and Clock Control) registers, specifically in the RCC_AHB1ENR register (for GPIO ports)
Example in C:
```RCC->AHB1ENR |= (1 << 0); // Clock enable for Port A (bit 0 - GPIOAEN)```
```RCC->AHB1ENR |= (1 << 1); // Enable clock for Port B (bit 1 - GPIOBEN)```

## 2. How registers are defined in code (CMSIS)
Header files define the physical addresses of registers in memory using C language structures:

The base address of the peripheral is defined in the memory map (e.g. GPIOA_BASE)

Individual port registers are grouped into the GPIO_TypeDef structure (contains items such as MODER, OTYPER, OSPEEDR, PUPDR, IDR, ODR, BSRR, etc.)

Access to the registers of a specific port is done via a pointer:

```GPIOA->MODER &= ~(0x00000003); // Access the MODER register of port A```

## 3. GPIO configuration
Each GPIO port (A, B, C...) has a set of 32-bit configuration registers available:
#### MODER (Mode Register): Defines the direction/mode of the pin (requires 2 bits for each pin):
- 00: Input (Default after reset)
- 01: General output
- 10: Alternative function (e.g. UART, SPI, PWM)
- 11: Analog mode (e.g. for ADC)
#### OTYPER (Output Type Register): Configures the output type (1 bit for each pin):
- 0: Push-Pull (Actively pushes to both logic 0 and logic 1)
- 1: Open-Drain (Suitable for buses like I2C, requires a pull-up resistor)
#### OSPEEDR (Output Speed ​​Register): Sets the switching speed of the pin:
- Options: 00 (Slow speed), 01 (Medium), 10 (Fast), 11 (High Speed)
#### PUPDR (Pull-Up/Pull-Down Register): Enables the integrated weak resistors (2 bits per pin):
- 00: No resistors (Floating)
- 01: Pull-Up (Connect to VDD)
- 10: Pull-Down (Connect to VSS)

## 4. Reading and writing data
#### IDR (Input Data Register): Used to read the physical state of the pins (Read-only)
Read: PAval = GPIOA->IDR;
#### ODR (Output Data Register): Directly write the output state (Read/Write)
Write: GPIOB->ODR |= 0x10; (Sets bit 4 to logical 1)
#### BSRR (Bit Set/Reset Register): 
- The best way to control the pins. Allows atomic writing without the risk of the process being interrupted by another task (ISR)
  - Writing 1 to the lower 16 bits sets the pin (Set to log. 1)
  - Writing 1 to the upper 16 bits resets the pin (Reset to log. 0)

##5. Configuring alternative functions (AF)
If you use the pin for an internal peripheral (e.g. USART2 on pins PA2/PA3 on the Nucleo board):
- Set MODER for the given pin to mode 10 (Alternate Function)

Using the AFRL (for pins 0-7) or AFRH (for pins 8-15) registers, select a specific alternative function (AF0 to AF15). Each pin has 4 bits reserved in these registers

## 6. Nucleo specifics (Hardware connections)
STM32 Nucleo boards have some pins hard-wired to the onboard hardware:
- Blue user button (B1 USER): Physically connected to pin PC13. There is a solder bridge SB17 on the board that provides this connection.
- User LED (LD2 / LD3): Often connected to pin PA5 (e.g. on NUCLEO-F401RE/F411RE/L152RE boards)
- Black button (B2 RESET): Directly connected to the NRST pin of the microcontroller to reset the device

