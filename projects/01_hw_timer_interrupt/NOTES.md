Now we'll replace `HAL_Delay()`, which calls a 1ms timer and checks it against
its argument, with a hardware timer.
HAL_Delay() itself doesn't hold the CPU with NOP instructions for N
miliseconds.

# TIM2 Mode and Configuration

Clock source: internal clock
## Counter settings
- prescaler (PSC) (0 --> 2^16-1)
At a CPU clock of 64 MHz,  say we want to have 1 ms ticks
we'd need to prescale 64,000 times
The formula is apparently
    `f_counter = f_timer / (PSC + 1)`
so set PSC=63,999

- Counter period (ARR = AutoReload Register) (0 --> 2^32)
Let's do 500ms which means 500 ticks
similar to PSC, `f_update = f_counter / (ARR + 1)`
so set ARR=499

- Counter Mode: up

** Every time the counter register counts up to 500, the compare against the
ARR register is true and it resets the counter to zero.

The TIM2 is a hardware timer unit on the uC made of flip-flops and counter logic.
Similar to the GPIOs, it's got volatile memory mapped registers in a struct.

```
typedef struct
{
    volatile uint32_t CR1; // Control register 1 -- controls basic counter behavior
                        // CEN (Counter EN), counting direction, etc
    volatile uint32_t CR2; // Control register 2 -- controls more counter behavior
    volatile uint32_t DIER;  // DMA/Interrupt Enable Register
    volatile uint32_t SR;  // Status Register -- shows events like counter
                           // wrapping back to zero or Update Interrupt Flag
    volatile uint32_t PSC;  // Prescaler value
    volatile uint32_t ARR;  // AutoReload Register value
    volatile uint32_t CNT;  // Actual counter value register
} TIM_TypeDef;
```

The base clock for a counter is defined in the .ioc file from CubeMX in the
clock distribution tree. It doesn't have to be the same as the CPU clock but
for us it is.

## What is the NVIC?
Nested Vector Interrupt Controller: A physical interceder between all the
hardware that generates Interrupt Requests (IRQs) and the CPU.
NVIC connects these IRQ generating peripherals like the DMA, Timers, ADCs, etc
into a priority order and decides which interrupts to service first.
It also goes to fetch the callback functions that should run when this
interrupt is generated from its vector table (a list of memory addresses
for those functions)

* The NVIC interrupt table needs to have TIM2 interrupts added to it.
  (which we had to do in CubeMX).
  This adds `TIM2_IRQhandler()` to that vector table and sets a priority for
  it. This is the **actual** interrupt service routine (ISR) that runs.

What we call at the application layer is an abstracted function:
`void HAL_TIM_PeriodElapsedCallback(TIM_HandleTypedef *htim)`
This is a general callback for all timers.
The HAL handles clearing/handling the innterrupt event and calls the
appropriate callback given the NVIC is set to respond to the timer.
^^
 This is a `__weak`ly defined function that gets overridden by our declaration
 to do what we want it to do, which in our case would be to check if the timer
 is TIM2 and then toggle GPIOE3.

## Code:

One new init function `MX_TIM2_Init();`

