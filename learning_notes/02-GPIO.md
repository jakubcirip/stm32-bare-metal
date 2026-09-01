# GPIO Overview for STM32F411xx

**STM32F411xx microcontrollers have ports GPIOA, GPIOB, GPIOC, GPIOD, GPIOE and GPIOH.
Each of these ports can independently control up to 16 physical pins.**

## 1. GPIO Clock Enable
In the STM32 architecture, all peripherals are disconnected from the clock after reset to save power.

If you try to write to the GPIO register without enabling the clock, the microcontroller will ignore your commands.

The clock signals for the GPIO ports are managed in the RCC_AHB1ENR register (RCC base address is 0x4002 3800, the register has an offset of 0x30).

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
