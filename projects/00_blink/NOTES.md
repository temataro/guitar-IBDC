Getting started with an STM32 dev board

Create a new project on CubeMX to generate a .ioc file that contains the
peripheral configurations I want for the uC.
Generate code to set it up the way I prescribed for GPIO, clocks, timers, UART,
etc
Open project in CubeIDE and write code in a `while(1)` block. OR  use the
`Makefile` I made to not even have to open CubeIDE.

HAL_GPIO_* functions just configure GPIO bank registers on the uC.
`static void MX_GPIO_Init(void)` takes a GPIO bank (like GPIOA, GPIOB, ...)
and sets parameters on their memory mapped registers:

internally something like this:
typedef struct
{
    volatile uint32_t MODER; // Mode Register
    volatile uint32_t OTYPER; // Output Type Register
    volatile uint32_t OSPEEDR; // Output speed register
    volatile uint32_t PUPDR; // Pull up/Pull Down register?
    ...
} GPIO_TypeDef;

Each bank holds 16 registers and each field in the struct is 2 bits wide.
Bits [1:0] in  the MODER register control GPIOX's first GPIO pin's mode.
