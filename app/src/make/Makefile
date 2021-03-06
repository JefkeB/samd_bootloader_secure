##############################################################################
BUILD = build
BIN = app

OPENOCD_CONFIG = ../openocd/openocd.cfg

STANDALONE ?= 1

LD_STANDALONE = ../linker/samd21j18.standalone.ld
LD_BOOTLOADER = ../linker/samd21j18.bootloader.ld

##############################################################################
.PHONY: all directory clean size

ccyellow=$(echo -e "\033[0;33m")
ccend=$(echo -e "\033[0m")

CC = arm-none-eabi-gcc
OBJCOPY = arm-none-eabi-objcopy
SIZE = arm-none-eabi-size

ifeq ($(OS), Windows_NT)
  MKDIR = gmkdir
else
  MKDIR = mkdir
endif
MKDIR = mkdir

CFLAGS += -W -Wall --std=gnu99 -Os
CFLAGS += -fno-diagnostics-show-caret
CFLAGS += -fdata-sections -ffunction-sections
CFLAGS += -funsigned-char -funsigned-bitfields
CFLAGS += -mcpu=cortex-m0plus -mthumb
CFLAGS += -MD -MP -MT $(BUILD)/$(*F).o -MF $(BUILD)/$(@F).d

LDFLAGS += -mcpu=cortex-m0plus -mthumb
LDFLAGS += -Wl,--gc-sections
ifeq ($(STANDALONE), 1)
$(info .)
$(info ### Building Standalone version ###)
$(info .)
LDFLAGS += -Wl,--script=$(LD_STANDALONE)
else
$(info .)
$(info *** Building Bootloader version ***)
$(info .)
LDFLAGS += -Wl,--script=$(LD_BOOTLOADER)
endif

INCLUDES += \
  -I../include \
  -I..

SRCS += \
  ../main.c \
  ../startup_samd21.c

DEFINES += \
  -D__SAMD21J18A__ \
  -DDONT_USE_CMSIS_INIT \
  -DF_CPU=8000000 

CFLAGS += $(INCLUDES) $(DEFINES)

OBJS = $(addprefix $(BUILD)/, $(notdir %/$(subst .c,.o, $(SRCS))))

all: directory $(BUILD)/$(BIN).elf $(BUILD)/$(BIN).hex $(BUILD)/$(BIN).bin size

$(BUILD)/$(BIN).elf: $(OBJS)
	@echo LD $@
	@$(CC) $(LDFLAGS) $(OBJS) $(LIBS) -o $@

$(BUILD)/$(BIN).hex: $(BUILD)/$(BIN).elf
	@echo OBJCOPY $@
	@$(OBJCOPY) -O ihex $^ $@

$(BUILD)/$(BIN).bin: $(BUILD)/$(BIN).elf
	@echo OBJCOPY $@
	@$(OBJCOPY) -O binary $^ $@

%.o:
	@echo CC $@
	@$(CC) $(CFLAGS) $(filter %/$(subst .o,.c,$(notdir $@)), $(SRCS)) -c -o $@

directory:
	@$(MKDIR) -p $(BUILD)

size: $(BUILD)/$(BIN).elf
	@echo size:
	@$(SIZE) -t $^

clean:
	@echo clean
	@-rm -rf $(BUILD)

program: $(BUILD)/$(BIN).hex
	openocd -f $(OPENOCD_CONFIG)

-include $(wildcard $(BUILD)/*.d)

