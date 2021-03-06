CROSS_COMPILE ?= arm-none-eabi-
CC := $(CROSS_COMPILE)gcc
AS := $(CROSS_COMPILE)as
CFLAGS = -fno-common -ffreestanding -O0 \
	 -gdwarf-2 -g3 -Wall -Werror \
	 -mcpu=cortex-m3 -mthumb \
	 -Wl,-Tos.ld -nostartfiles \

TARGET = os.bin
all: $(TARGET)

$(TARGET): os.c startup.c context_switch.S syscall.S
	$(CC) $(CFLAGS) $^ -o os.elf
	$(CROSS_COMPILE)objcopy -Obinary os.elf os.bin
	$(CROSS_COMPILE)objdump -S os.elf > os.list

qemu: $(TARGET)
	@qemu-system-arm -M ? | grep stm32-p103 >/dev/null || exit
	@echo "Press Ctrl-A and then X to exit QEMU"
	@echo
	qemu-system-arm -M stm32-p103 -nographic -kernel os.bin

clean:
	rm -f *.o *.elf *.bin *.list

debug:
	killall openocd || true
	openocd \
		-s "/usr/share/openocd/scripts" \
		-f board/st_nucleo_f103rb.cfg \
		--log_output openocd.log \
		-c "init" \
	    -c "reset halt" \
		&
	sleep 1
	arm-none-eabi-gdb -tui \
		-ex "set pagination off" \
		-ex "target remote localhost:3333" \
		-ex "layout split" \
		-ex "layout regs" \
		-ex "tui reg all" \
		-ex "b os.c:74" \
		os.elf
