# =============================================================================
# Makefile — Build and flash STM32H723 project from command line
#
# Wraps the CubeIDE-generated Debug/makefile for CLI-based development.
# Auto-detects DFU devices on USB so you never have to hardcode port numbers.
#
# Copyright (c) 2026 Temesgen Ataro <temesgen.ataro@tii.ae>
# SPDX-License-Identifier: Apache-2.0
# =============================================================================

# --- Configuration -----------------------------------------------------------

PROGRAMMER_CLI ?= /home/tem/STMicroelectronics/STM32Cube/STM32CubeProgrammer/bin/STM32_Programmer_CLI
BUILD_DIR      := Debug
ELF            := $(BUILD_DIR)/01_hw_timer_interrupt.elf
DFU_VID_PID    := 0483:df11
SRC_DIRS       := Core Drivers
# Override with SUDO= after the udev rule works without elevated access.
SUDO           ?= sudo

# --- Colors ------------------------------------------------------------------

ifneq ($(TERM),)
  GREEN  := \033[32m
  RED    := \033[31m
  YELLOW := \033[33m
  BOLD   := \033[1m
  RESET  := \033[0m
endif

# --- Internal helpers --------------------------------------------------------

# Detect the USB port name by asking STM32_Programmer_CLI to list DFU devices.
# CubeProgrammer lists DFU devices before its J-Link section. Extract its
# `USB1`, `USB2`, ... device index without relying on a version-specific
# rendering of the USB VID:PID.
detect_port = $(strip $(shell $(SUDO) $(PROGRAMMER_CLI) -l 2>/dev/null | awk '/DFU Interface/{dfu=1; next} /J-Link Interface/{dfu=0} dfu' | grep -oE 'USB[0-9]+' | head -1))

# Check for DFU using the same tool and privilege level used to flash.
check_dfu = @port='$(call detect_port)'; \
	test -n "$$port" || \
	(echo "$(RED)$(BOLD)Error:$(RESET) No STM32 DFU device found ($(DFU_VID_PID))."; \
	echo "Set BOOT0 high, reset the board, then reconnect its USB data cable."; \
	exit 1)

# Fail early with a clear message if the ARM GCC toolchain is missing.
check_toolchain = @command -v arm-none-eabi-gcc >/dev/null 2>&1 || \
	(echo "$(RED)$(BOLD)Error:$(RESET) 'arm-none-eabi-gcc' not found on PATH."; \
	echo "Install it, or add CubeIDE's bundled toolchain bin/ dir to PATH:"; \
	echo "  sudo apt install gcc-arm-none-eabi"; \
	exit 1)

# --- Targets -----------------------------------------------------------------

check-dfu:
	$(check_dfu)

check-toolchain:
	$(check_toolchain)

.PHONY: all flash flash-all erase clean size disasm watch info help check-dfu check-toolchain

## Build the project via the CubeIDE-generated makefile
all: check-toolchain
	@echo "$(GREEN)Building $(BOLD)$(ELF)$(RESET)..."
	$(MAKE) -C $(BUILD_DIR) all -j$$(nproc)
	@test -f $(ELF) || \
	(echo "$(RED)$(BOLD)Build failed:$(RESET) no $(ELF) produced. Scroll up for compile errors." && exit 1)
	@echo "$(GREEN)$(BOLD)Build complete.$(RESET)"
	@$(MAKE) --no-print-directory size

## Flash the last-built ELF (does NOT rebuild first)
flash: check-dfu
	@port=$(call detect_port); \
	echo "$(YELLOW)Flashing$(RESET) $(ELF) $(YELLOW)to $$port...$(RESET)"; \
	$(SUDO) $(PROGRAMMER_CLI) -c port=$$port -w $(ELF) -v -rst
	@echo "$(GREEN)$(BOLD)Flash + reset complete.$(RESET)"

## Build and flash in one step
flash-all: all flash

## Erase the entire chip
erase: check-dfu
	@port=$(call detect_port); \
	echo "$(YELLOW)Erasing chip via $$port...$(RESET)"; \
	$(SUDO) $(PROGRAMMER_CLI) -c port=$$port -e all
	@echo "$(GREEN)$(BOLD)Erase complete.$(RESET)"

## Remove all build artifacts
clean:
	$(MAKE) -C $(BUILD_DIR) clean
	@echo "$(GREEN)Clean complete.$(RESET)"

## Show firmware size breakdown
size:
	@test -f $(ELF) || \
	(echo "$(RED)$(BOLD)No ELF found.$(RESET) Run $(BOLD)make$(RESET) first." && exit 1)
	@echo ""
	@echo "$(BOLD)Firmware size:$(RESET)"
	@arm-none-eabi-size $(ELF)

## Generate full disassembly listing
disasm:
	@test -f $(ELF) || \
	(echo "$(RED)$(BOLD)No ELF found.$(RESET) Run $(BOLD)make$(RESET) first." && exit 1)
	@arm-none-eabi-objdump -d -S $(ELF) > $(BUILD_DIR)/01_hw_timer_interrupt.disasm
	@echo "$(GREEN)Disassembly written to $(BUILD_DIR)/01_hw_timer_interrupt.disasm$(RESET)"

## Rebuild on source change (uses inotifywait)
watch:
	@echo "$(YELLOW)Watching $(SRC_DIRS)/ for changes... Press Ctrl-C to stop.$(RESET)"
	@while true; do \
		$(MAKE) --no-print-directory all; \
		inotifywait -r -q -e modify,create,delete $(SRC_DIRS) 2>/dev/null; \
	done

## Show project metadata
info:
	@echo "$(BOLD)Project:$(RESET)     01_hw_timer_interrupt"
	@echo "$(BOLD)MCU:$(RESET)        STM32H723VGTx"
	@echo "$(BOLD)Toolchain:$(RESET)  GNU Tools for STM32"
	@echo "$(BOLD)ELF:$(RESET)        $(ELF)"
	@echo "$(BOLD)DFU port:$(RESET)   $(or $(call detect_port),not connected)"

## List available targets
help:
	@echo "$(BOLD)Usage:$(RESET)"
	@echo "  make              Build the project"
	@echo "  make flash        Flash the last build (auto-detects USB port)"
	@echo "  make flash-all    Build + flash"
	@echo "  make erase        Erase the entire chip"
	@echo "  make clean        Remove build artifacts"
	@echo "  make size         Show firmware size"
	@echo "  make disasm       Generate disassembly listing"
	@echo "  make watch        Rebuild on source change"
	@echo "  make info         Show project metadata"
	@echo "  make help         This message"
