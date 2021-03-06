#
#  Example makefile to build an MSP432 LP TI-RTOS "reusable configuration";
#  a fixed configuration of TI-RTOS that can be consumed by applications as
#  a 100% self contained set of libraries and headers.
#
#  This configuration requires the installation directory of each of the
#  following prerequisite components:
#      CCROOT  - the GCC arm compiler (version 4.8 or above)
#      XDCROOT - XDCtools (version 3.30.05 or above)
#      TIRTOS  - TI-RTOS for CC26xx
#      EMTROOT - the EMT packages repo
#

# include definitions of the macros described above
include tools.mak

# define the XDC package path based on the settings above
SYSBIOS ?= $(wildcard $(TIRTOS)/products/bios_*)

XDCPATH = $(TIRTOS)/packages;$(SYSBIOS)/packages;$(EMTROOT)

PLATFORM = ti.platforms.emt26xx
TARGET   = gnu.targets.arm.M3
CCOPTS   = --compileOptions "-gdwarf-3 -gstrict-dwarf -fno-exceptions -I$(DRVLIB.cc26xx)"

VARIANTS  = $(EMTROOT)/ti/runtime/wiring/cc26xx/variants

ifneq (,$(wildcard $(XDCROOT)/bin/rm.exe))
    RMDIR = $(XDCROOT)/bin/rm -rf
else
    RMDIR = rm -rf
endif

all: .debug closure

# output all prerequisites used to create the configuration
.debug:
	@echo "  ---- imports ----"
	@echo "    CCROOT  = $(CCROOT)"
	@echo "    XDCROOT = $(XDCROOT)"
	@echo "    TIRTOS  = $(lastword $(subst /, ,$(TIRTOS)))"
	@echo "    SYSBIOS = $(SYSBIOS)"
	@echo "    EMTROOT = $(EMTROOT)"
	@echo "    XDCPATH = $(XDCPATH)"
	@echo "    DRVLIB  = $(DRVLIB.cc26xx)"
	@echo "  -----------------"
	@echo ""

# create a single directory containing _everything_ referenced by configPkg
closure: configPkg
	@$(RMDIR) $@
	@echo xs xdc.tools.closure -d $@ $< ...
	$(XDCROOT)/xs xdc.tools.closure -d $@ -i ti.drivers.bsp -i ti.runtime.driverlib.cc26xx $<

	$(XDCROOT)/bin/xdccpp $(wildcard $(VARIANTS)/*) $@

	@echo WARNING: copying ti.drivers headers to $@ from $(EMTROOT)/ti/drivers
	@cp $(EMTROOT)/ti/drivers/gpio/* $@/ti/drivers/gpio
	@cp $(EMTROOT)/ti/drivers/pwm/* $@/ti/drivers/pwm

	@echo patching $@ ...
	@$(CURDIR)/../patch.ksh $@ ./src $<

# create a fixed configuration of TI-RTOS from energia.cfg
configPkg: energia.cfg
	@echo xs xdc.tools.configuro -o $@ $< ...
	$(XDCROOT)/xs --xdcpath="$(XDCPATH)" xdc.tools.configuro -o $@ -t $(TARGET) $(CCOPTS) -p $(PLATFORM) -r release -c $(CCROOT) $<

# remove all generated files/directories
clean:
	@$(RMDIR) closure configPkg src closure.zip
