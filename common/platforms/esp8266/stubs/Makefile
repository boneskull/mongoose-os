#
# Copyright (c) 2015 Cesanta Software Limited
# All rights reserved
#

STUB = stub_hello.c
LIBS =
PARAMS =
PORT = /dev/ttyUSB0

BUILD_DIR = .build
COMMON_STUB_DIR = ../../esp
STUB_ELF = $(BUILD_DIR)/$(patsubst %.c,%.elf,$(notdir $(STUB)))
STUB_JSON ?= $(BUILD_DIR)/$(patsubst %.c,%.json,$(notdir $(STUB)))
SDK = docker.cesanta.com/esp8266-build-oss:2.0.0-r7
XT_CC = xtensa-lx106-elf-gcc

.PHONY: all clean run wrap

all: $(STUB_ELF)

$(STUB_ELF): $(STUB) $(LIBS)
	@echo "  CC   $^ -> $@"
	@[ -d $(BUILD_DIR) ] || mkdir $(BUILD_DIR)
	@docker run --rm -i -v $(CURDIR)/../../../..:/src $(SDK) //bin/bash -c \
    "cd /src/common/platforms/esp8266/stubs && \
     $(XT_CC) -std=c99 -Wall -Werror -Os -DESP8266 \
         -mtext-section-literals -mlongcalls -nostdlib -fno-builtin \
         -I. -I/src/common/platforms/esp \
         -I/opt/Espressif/ESP8266_SDK \
         -Wl,-static -ffunction-sections -Wl,--gc-sections \
         -Tstub.ld -o $@ $^"

wrap: $(STUB_JSON)

$(STUB_JSON): $(STUB_ELF) $(COMMON_STUB_DIR)/esptool.py
	@echo "  WRAP $< -> $@"
	@docker run --rm -i -v $(CURDIR)/../../../..:/src $(SDK) //bin/bash -c \
    "cd /src/common/platforms/esp8266/stubs && \
     $(COMMON_STUB_DIR)/esptool.py wrap_stub $<" > $@

run: $(STUB_JSON)
	@echo "  RUN  $< $(PARAMS) -> $(PORT)"
	@docker run --rm -i --privileged -v $(CURDIR)/../../../..:/src $(SDK) //bin/bash -c \
    "cd /src/common/platforms/esp8266/stubs && \
     $(COMMON_STUB_DIR)/esptool.py --port $(PORT) run_stub $< $(PARAMS)"

clean:
	@rm -rf $(BUILD_DIR)
