# GPIO Overview for STM32F411xx

**STM32F411xx microcontrollers have ports GPIOA, GPIOB, GPIOC, GPIOD, GPIOE and GPIOH.
Each of these ports can independently control up to 16 physical pins.**

## 1. GPIO Clock Enable
In the STM32 architecture, all peripherals are disconnected from the clock after reset to save power. \
If you try to write to the GPIO register without enabling the clock, the microcontroller will ignore your commands. \
The clock signals for the GPIO ports are managed in the RCC_AHB1ENR register (RCC base address is 0x4002 3800, the register has an offset of 0x30). \

The individual ports correspond to specific bits in RCC_AHB1ENR:
- Bit 0 (GPIOAEN): Port A
- Bit 1 (GPIOBEN): Port B
- Bit 2 (GPIOCEN): Port C
- Bit 3 (GPIODEN): Port D
- Bit 4 (GPIOEEN): Port E
- Bit 7 (GPIOHEN): Port H

Example in C: \
```RCC->AHB1ENR |= (1U<<0); // Enable clock for GPIOA (bit 0) [9]``` \
```RCC->AHB1ENR |= (1U<<2); // Enable clock for GPIOC (bit 2) [10]```

## 2. Boundary Addresses
If we access the registers directly (bare-metal access), the base addresses of individual ports in memory are as follows:
- GPIOA: 0x4002 0000
- GPIOB: 0x4002 0400
- GPIOC: 0x4002 0800
- GPIOD: 0x4002 0C00
- GPIOE: 0x4002 1000
- GPIOH: 0x4002 1C00

These addresses are mapped in the CMSIS libraries using the **GPIO_TypeDef** structure based on pointers.

## 3. Main GPIO configuration registers
Each port is set separately using a set of 32-bit registers:
#### GPIOx_MODER (Mode Register, offset 0x00): Specifies the pin mode (requires 2 bits per pin):
- 00: Input (default after reset for most pins)
- 01: General purpose output (general purpose output)
- 10: Alternate function (alternative function for peripherals - e.g. SPI, I2C, USART)
- 11: Analog mode (analog mode for ADC / low power)

_Note: Port A has a reset value of 0xA800 0000 and Port B 0x0000 0280 due to the default configuration of the debug pins (JTAG/SWD)_

#### GPIOx_OTYPER (Output Type Register, offset 0x04):
Sets the output circuit (requires 1 bit per pin):
- 0: Push-Pull (actively pushes the pin to both logic 0 and logic 1)
- 1: Open-Drain (output transistor grounds the pin to logic 0, requires a pull-up for logic 1) resistor)

#### GPIOx_OSPEEDR (Output Speed ​​Register, offset 0x08):
Specifies the maximum output switching frequency (requires 2 bits per pin):
- 00: Low speed
- 01: Medium speed
- 10: Fast speed
- 11: High speed

#### GPIOx_PUPDR (Pull-up/Pull-down Register, offset 0x0C):
Enables the integrated weak resistors (requires 2 bits per pin):
- 00: No resistors (Floating)
- 01: Pull-Up (connects pin to internal power supply VDD)
- 10: Pull-Down (connects pin to ground VSS)
- 11: Reserved (not used)

