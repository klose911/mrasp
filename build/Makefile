# uname_S := $(shell sh -c 'uname -s 2>/dev/null || echo not')

TOOLCHAIN = arm-none-eabi
CC = $(TOOLCHAIN)-gcc
OBJCOPY = $(TOOLCHAIN)-objcopy
GDB = $(TOOLCHAIN)-gdb

# ifeq ($(OS),Windows_NT)
# 	CC = $(TOOLCHAIN)-gcc.exe
# 	OBJCOPY = $(TOOLCHAIN)-objcopy.exe
# 	GDB = $(TOOLCHAIN)-gdb.exe
# endif

# Set any constants based on the raspberry pi model.  Version 1 has some differences to 2 and 3
ifeq ($(RASPI_MODEL),1)
	CPU = arm1176jzf-s
	DIRECTIVES = -D MODEL_1
	ARCHDIR = model1
else
	CPU = cortex-a7
	ARCHDIR = model2
endif

CFLAGS= -mcpu=$(CPU) -fpic -ffreestanding $(DIRECTIVES) -g
CSRCFLAGS= -O2 -Wall -Wextra
LFLAGS= -ffreestanding -O2 -nostdlib

# Location of the files
KER_SRC = ../src/kernel
KER_HEAD = ../include
COMMON_SRC = ../src/common
OBJ_DIR = objects
KERSOURCES = $(wildcard $(KER_SRC)/*.c)
KERSOURCES += $(wildcard $(KER_SRC)/$(ARCHDIR)/*.c)
COMMONSOURCES = $(wildcard $(COMMON_SRC)/*.c)
ASMSOURCES = $(wildcard $(KER_SRC)/*.S)
OBJECTS = $(patsubst $(KER_SRC)/%.c, $(OBJ_DIR)/%.o, $(KERSOURCES))
OBJECTS += $(patsubst $(COMMON_SRC)/%.c, $(OBJ_DIR)/%.o, $(COMMONSOURCES))
OBJECTS += $(patsubst $(KER_SRC)/%.S, $(OBJ_DIR)/%.o, $(ASMSOURCES))
HEADERS = $(wildcard $(KER_HEAD)/*.h)

IMG_NAME=kernel

build: $(OBJECTS) $(HEADERS)
	$(CC) -T linker.ld -o $(IMG_NAME).elf $(LFLAGS) $(OBJECTS)
	$(OBJCOPY) $(IMG_NAME).elf -O binary $(IMG_NAME).img

$(OBJ_DIR)/%.o: $(KER_SRC)/%.c
	mkdir -p $(@D)
	$(CC) $(CFLAGS) -I$(KER_SRC) -I$(KER_HEAD) -c $< -o $@ $(CSRCFLAGS)

$(OBJ_DIR)/%.o: $(KER_SRC)/$(ARCHDIR)/%.c
	mkdir -p $(@D)
	$(CC) $(CFLAGS) -I$(KER_SRC) -I$(KER_HEAD) -c $< -o $@ $(CSRCFLAGS)

$(OBJ_DIR)/%.o: $(KER_SRC)/%.S
	mkdir -p $(@D)
	$(CC) $(CFLAGS) -I$(KER_SRC) -c $< -o $@

$(OBJ_DIR)/%.o: $(COMMON_SRC)/%.c
	mkdir -p $(@D)
	$(CC) $(CFLAGS) -I$(KER_SRC) -I$(KER_HEAD) -c $< -o $@ $(CSRCFLAGS)

clean:
	rm -rf $(OBJ_DIR)
	rm $(IMG_NAME).elf
	rm $(IMG_NAME).img


QEMU_SUPPORT_RASPI2B := $(shell sh -c 'qemu-system-arm --machine help | grep raspi2b ')
ifeq ($(strip $(QEMU_SUPPORT_RASPI2B)), )
	QEMU_MACHINE := raspi2
	QEMU_MEMORRY := 128
else
	QEMU_MACHINE := raspi2b
	QEMU_MEMORRY := 1024
endif 

run: build
	qemu-system-arm -m $(QEMU_MEMORRY) -no-reboot -M $(QEMU_MACHINE) -serial stdio -kernel kernel.elf

dbg:
	$(GDB) kernel.elf

dbgrun: build gdbinit
	qemu-system-arm -m $(QEMU_MEMORRY) -no-reboot -M $(QEMU_MACHINE) -serial stdio -kernel kernel.elf -S -s

.PHONY: gdbinit
gdbinit:
	echo "target remote localhost:1234" > .gdbinit
	echo "break kernel_main" >> .gdbinit
