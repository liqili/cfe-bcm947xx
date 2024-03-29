#
# Makefile for compressed self-booting CFE on Broadcom BCM947XX boards
#
# Copyright (C) 2012, Broadcom Corporation
# All Rights Reserved.
# 
# This is UNPUBLISHED PROPRIETARY SOURCE CODE of Broadcom Corporation;
# the contents of this file may not be disclosed to third parties, copied
# or duplicated in any form, in whole or in part, without the prior
# written permission of Broadcom Corporation.
#
# $Id: Makefile 367718 2012-11-09 03:57:10Z $
#

CROSS_COMPILE	:= $(TOOLPREFIX)
CC		:= $(GCC)
LD		:= $(GLD)
LOADADDR	:= $(CFG_TEXT_START)

# The self-decompresor uses the minimal OSL, get rid of _CFE_ and vxworks
CFLAGS		:= $(subst -D_CFE_,,$(CFLAGS)) -fdata-sections
CFLAGS		+= -D_CFEZ_ -D_MINOSL_ -DBCMBUSTYPE=SI_BUS -Uvxworks -U__vxworks

# Default mips
ifeq ($(ARCH),)
ARCH := mips
endif

OBJCOPY		:= $(CROSS_COMPILE)objcopy -O binary -R .reginfo -R .note -R .comment -R .mdebug -S
OBJCOPYSREC	:= $(CROSS_COMPILE)objcopy -O srec -R .reginfo -R .note -R .comment -R .mdebug -S

SYSTEM		:= ../cfe

vpath %.c $(SRCBASE)/shared $(SRCBASE)/cfe/cfe/arch/arm/board/bcm947xx/src
vpath %.S $(SRCBASE)/shared
vpath %.lds.in $(SRCBASE)/shared $(SRCBASE)/tools/misc/lzma_src/C

CFLAGS		+= -DLOADADDR=$(LOADADDR)
CFLAGS		+= -ffunction-sections $(call check_gcc, -fvtable-gc, )

# Note: siutils.o appeared seperately on purpose to solve post reference problem of GCC
OBJS-mips = boot.o sisdram.o siutils.o hndmips.o hndpmu.o
OBJS-arm = bootarm.o  aisdram_ca9.o siutils.o hndarm_ca9.o \
	   pcieinit_ca9.o aiutils.o

ifeq ($(strip $(CFG_SHMOO)), 1)
OBJS-arm += aisdram-ca9.o hndsoc.o \
	    ddr40_phy_init.o platform.o shmoo_DDR3.o tinymt32.o
endif

OBJECTS := $(OBJS-$(ARCH))
OBJECTS += load.o hndchipc.o min_osl.o bcmutils.o \
	   bcmstdlib.o sromstubs.o nvramstubs.o

ifeq ($(ARCH), arm)
  ELF32_LITTLE := elf32-littlearm
  ELF32_BIG    := elf32-bigarm
  CFLAGS       += -DBCMCHIPTYPE=3
  ifeq ($(strip $(CFG_SHMOO)), 1)
    TEXT_START   := 0xc0000000
    CFLAGS	 += -DCFG_SHMOO=1
    CFEZ_LDS_IN  := cfez_arm_shmoo.lds.in
  else
    TEXT_START   := 0x00001000
    CFEZ_LDS_IN  := cfez_arm.lds.in
  endif
else
  CFEZ_LDS_IN  := hndrte.lds.in
  ELF32_LITTLE := elf32-tradlittlemips
  ELF32_BIG    := elf32-bigmips

  ifeq ($(strip $(CFG_MINIMAL_SIZE)), 1)
    ifeq ($(strip $(CFG_GMAC)), 0)
      CFLAGS   += -DBCMCHIPTYPE=0
      OBJECTS  += sbsdram.o sbutils.o
    else
      CFLAGS   += -DBCMCHIPTYPE=1
      OBJECTS  += aisdram.o aiutils.o
    endif
  else
    OBJECTS    += aisdram.o aiutils.o sbsdram.o sbutils.o
  endif

  # Link at 3 MB offset in RAM
  TEXT_START	:= 0x80001000
endif

ifeq ($(strip $(CFG_NFLASH)), 1)
OBJECTS		+= nflash.o
endif

# Default to lzma_4k
COMPRESS	:= lzma_4k e

ifneq ($(findstring lzma_4k,$(COMPRESS)),)
CFLAGS		+= -DUSE_LZMA -DBCMLZMA
else
COMPRESS	:= cat
endif

ifeq ($(RTAC68U), 1)
ifeq ($(BCM470XC0), 1)
	CFE_BIN_FILE := cfe_rt-ac68u_5003_c0.bin
else ifeq ($(BCM470XC0), 2)
	CFE_BIN_FILE := cfe_rt-ac68u_v2_5003_c0.bin
else ifeq ($(PATYPE), 0)
	CFE_BIN_FILE := cfe_rt-ac68u.bin
else ifeq ($(PATYPE), R6250)
	CFE_BIN_FILE := cfe_rt-ac68u-r6250.bin
else ifeq ($(PATYPE), 5023)
	CFE_BIN_FILE :=	cfe_rt-ac68u_v2_5023.bin
else ifeq ($(PATYPE), 85402)
	CFE_BIN_FILE := cfe_rt-ac68u_v2_85402.bin
else ifeq ($(PATYPE), 5003)
	CFE_BIN_FILE := cfe_rt-ac68u_v2_5003.bin
endif
endif
ifeq ($(RT4GAC68U), 1)
	CFE_BIN_FILE := cfe_4g-ac68u_5003_c0.bin
endif


all: cfez.bin

# Don't build dependencies, this may die if $(CC) isn't gcc
dep:

cfez.bin: cfez
	$(OBJCOPY) $< $@
	$(OBJCOPYSREC) $< cfez.srec

ifeq ($(DSLAC68U), 1)
ifeq ($(JDM), 1)
	../../../../../ctools/nvserial -i cfez.bin -o cfe_dsl-ac68u_jdm.bin -s 00 dsl-ac68u_jdm_nvram.txt
	cp cfe_dsl-ac68u_jdm.bin ../../../..
else
	../../../../../ctools/nvserial -i cfez.bin -o cfe_dsl-ac68u.bin -s 00 dsl-ac68u_nvram.txt
	cp cfe_dsl-ac68u.bin ../../../..
endif
endif

ifeq ($(RTAC87U), 1)
	../../../../../ctools/nvserial -i cfez.bin -o cfe_rt-ac87u.bin -s 00 rt-ac87u_nvram.txt
	cp cfe_rt-ac87u.bin ../../../..
	@if [ "$(CFEZ_MAXSIZE)" != "" ]; then \
		if [ "`wc -c < cfe_rt-ac87u.bin`" -gt "$(CFEZ_MAXSIZE)" ]; then \
			echo "*** ERROR *** : cfe_rt-ac87u.bin bootrom image size: `wc -c < cfe_rt-ac87u.bin` exceeds limit: $(CFEZ_MAXSIZE)"; \
		fi ; \
	fi
else
ifeq ($(RTAC68U), 1)
ifeq ($(BCM470XC0), 1)
	../../../../../ctools/nvserial -i cfez.bin -o cfe_rt-ac68u_5003_c0.bin -s 00 rt-ac68u_5003_c0_nvram.txt
	cp cfe_rt-ac68u_5003_c0.bin ../../../..
else ifeq ($(BCM470XC0), 2)
	../../../../../ctools/nvserial -i cfez.bin -o cfe_rt-ac68u_v2_5003_c0.bin -s 00 rt-ac68u_v2_5003_c0_nvram.txt
	cp cfe_rt-ac68u_v2_5003_c0.bin ../../../..
else
ifeq ($(PATYPE), 0)
	../../../../../ctools/nvserial -i cfez.bin -o cfe_rt-ac68u.bin -s 00 rt-ac68u_nvram.txt
	cp cfe_rt-ac68u.bin ../../../..
else ifeq ($(PATYPE), R6250)
	../../../../../ctools/nvserial -i cfez.bin -o cfe_rt-ac68u-r6250.bin -s 00 rt-ac68u_nvram_r6250.txt
	cp cfe_rt-ac68u.bin ../../../..
else ifeq ($(PATYPE), 5023)
	../../../../../ctools/nvserial -i cfez.bin -o cfe_rt-ac68u_v2_5023.bin -s 00 rt-ac68u_v2_5023_nvram.txt
	cp cfe_rt-ac68u_v2_5023.bin ../../../..
else ifeq ($(PATYPE), 85402)
	../../../../../ctools/nvserial -i cfez.bin -o cfe_rt-ac68u_v2_85402.bin -s 00 rt-ac68u_v2_85402_nvram.txt
	cp cfe_rt-ac68u_v2_85402.bin ../../../..
else ifeq ($(PATYPE), 5003)
	../../../../../ctools/nvserial -i cfez.bin -o cfe_rt-ac68u_v2_5003.bin -s 00 rt-ac68u_v2_5003_nvram.txt
	cp cfe_rt-ac68u_v2_5003.bin ../../../..
else ifeq ($(PATYPE), 47094)
	../../../../../ctools/nvserial -i cfez.bin -o cfe_rt-ac88b.bin -s 00 rt-ac88b_nvram.txt
	cp cfe_rt-ac88b.bin ../../../..
endif
endif
else
ifeq ($(RTN18U), 1)
		../../../../../ctools/nvserial -i cfez.bin -o cfe_rt-n18u.bin -s 00 rt-n18u_nvram.txt
		cp cfe_rt-n18u.bin ../../../..
endif
ifeq ($(BUILD_NAME), RP-AC68U)
		../../../../../ctools/nvserial -i cfez.bin -o cfe_rp-ac68u.bin -s 00 rp-ac68u_nvram.txt
		cp cfe_rp-ac68u.bin ../../../..
endif
endif
endif
ifeq ($(RT4GAC68U), 1)
	../../../../../ctools/nvserial -i cfez.bin -o cfe_4g-ac68u_5003_c0.bin -s 00 4g-ac68u_5003_c0_nvram.txt
	cp cfe_4g-ac68u_5003_c0.bin ../../../..
endif

ifeq ($(DSLAC68U), 1)
	@if [ "$(CFEZ_MAXSIZE)" != "" ]; then \
		if [ "`wc -c < cfe_dsl-ac68u.bin`" -gt "$(CFEZ_MAXSIZE)" ]; then \
			echo "*** ERROR *** : cfe_dsl-ac68u.bin bootrom image size: `wc -c < cfe_dsl-ac68u.bin` exceeds limit: $(CFEZ_MAXSIZE)"; \
		fi ; \
	fi
endif
ifeq ($(RTAC68U), 1)
	@if [ "$(CFEZ_MAXSIZE)" != "" ]; then \
		if [ "`wc -c < $(CFE_BIN_FILE)`" -gt "$(CFEZ_MAXSIZE)" ]; then \
			echo "*** ERROR *** : cfe_rt-ac68u.bin bootrom image size: `wc -c < cfe_rt-ac68u.bin` exceeds limit: $(CFEZ_MAXSIZE)"; \
		fi ; \
	fi
else
ifeq ($(RTN18U), 1)
	@if [ "$(CFEZ_MAXSIZE)" != "" ]; then \
		if [ "`wc -c < cfe_rt-n18u.bin`" -gt "$(CFEZ_MAXSIZE)" ]; then \
			echo "*** ERROR *** : cfe_rt-n18u.bin bootrom image size: `wc -c < cfe_rt-n18u.bin` exceeds limit: $(CFEZ_MAXSIZE)"; \
		fi ; \
	fi
endif
ifeq ($(BUILD_NAME), RP-AC68U)
	@if [ "$(CFEZ_MAXSIZE)" != "" ]; then \
		if [ "`wc -c < cfe_rp-ac68u.bin`" -gt "$(CFEZ_MAXSIZE)" ]; then \
			echo "*** ERROR *** : cfe_rp-ac68u.bin bootrom image size: `wc -c < cfe_rp-ac68u.bin` exceeds limit: $(CFEZ_MAXSIZE)"; \
		fi ; \
	fi
endif
endif
ifeq ($(RT4GAC68U), 1)
	@if [ "$(CFEZ_MAXSIZE)" != "" ]; then \
		if [ "`wc -c < $(CFE_BIN_FILE)`" -gt "$(CFEZ_MAXSIZE)" ]; then \
			echo "*** ERROR *** : cfe_4g-ac68u.bin bootrom image size: `wc -c < cfe_4g-ac68u.bin` exceeds limit: $(CFEZ_MAXSIZE)"; \
		fi ; \
	fi
endif

# Link the loader and the kernel binary together
cfez: cfez.lds $(OBJECTS) piggy.o
	$(LD) -static -Map cfez.map --gc-sections -no-warn-mismatch -T cfez.lds -o $@ $(OBJECTS) $(LDLIBS) piggy.o

cfez.lds: $(CFEZ_LDS_IN) Makefile
	sed -e s/TEXT_START/$(TEXT_START)/ \
	    -e s/TARGET_ARCH/$(ARCH)/ < $< > $@

# Create a linkable version of the (possibly compressed) kernel binary
piggy.o: piggz piggy.lds

ifeq ($(CFG_LITTLE), 1)
	$(LD) -no-warn-mismatch -T piggy.lds -r -o $@ -b binary piggz -b $(ELF32_LITTLE)
else
	$(LD) -no-warn-mismatch -T piggy.lds -r -o $@ -b binary piggz -b $(ELF32_BIG)
endif

piggy.lds:
	@echo "SECTIONS { .data : { input_len = .; LONG(input_data_end - input_data) input_data = .; *(.data) input_data_end = .; }}" > $@

piggz: piggy
ifneq ($(findstring lzma_4k,$(COMPRESS)),)
	$(COMPRESS) $< $@
else
	$(COMPRESS) $< > $@
endif

piggy: $(SYSTEM)
	$(OBJCOPY) $< $@

mrproper: clean

clean:
	rm -f cfez cfez.bin cfez.srec piggz piggy *.lds *.o *.map

%.o : %.S
	$(GCC) $(ASFLAGS) $(CFLAGS) -o $@ $<
