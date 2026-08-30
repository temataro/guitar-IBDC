# STM32H723 DSP Learning Plan
## Goal: Basic Working Proficiency in STM32 Embedded DSP

### Hardware
- **MCU board:** WeAct Studio MiniSTM32H723
- **MCU:** STM32H723VGT6
  - ARM Cortex-M7
  - Up to 550 MHz
  - Hardware FPU
  - DSP instructions
  - DMA
  - ADCs
  - Timers
  - SPI / I2S / SAI
- **Eventual audio codec:** AK4558EN
- **Application/capstone:** real-time guitar DSP / looper pedal
- **Development OS:** Linux

### Toolchain
- **STM32CubeMX:** peripheral/pin/clock configuration and code generation
- **STM32CubeIDE:** editing, compilation, debugging
- **arm-none-eabi-gcc:** actual compiler/toolchain
- **GNU Make:** build automation
- **STM32CubeProgrammer CLI:** DFU flashing
- **ST-Link + SWD:** preferred debugger/programmer
- **CMSIS / CMSIS-DSP:** ARM MCU definitions and optimized DSP library

---

# 1. Overall Objective

The objective is **not** to become familiar with every STM32 peripheral.

The objective is to become capable of independently building and understanding this system:

```text
analog signal
    ↓
AK4558 ADC
    ↓
SAI/I2S
    ↓
DMA
    ↓
ping-pong audio buffers
    ↓
real-time DSP
    ↓
DMA
    ↓
SAI/I2S
    ↓
AK4558 DAC
    ↓
analog output
