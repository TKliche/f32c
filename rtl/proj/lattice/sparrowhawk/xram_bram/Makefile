
PROJ_FILE := $(shell ls *.ldf | head -1)
PROJ_NAME := $(shell fgrep default_implementation ${PROJ_FILE} | cut -d'"' -f 4)
IMPL_NAME := $(shell fgrep default_implementation ${PROJ_FILE} | cut -d'"' -f 8)
IMPL_DIR := $(shell fgrep default_strategy ${PROJ_FILE} | cut -d'"' -f 4)

DIAMOND_BASE := /usr/local/diamond
DIAMOND_BIN :=  $(shell find ${DIAMOND_BASE}/ -maxdepth 2 -name bin | sort -rn | head -1)
DIAMONDC := $(shell find ${DIAMOND_BIN}/ -name diamondc)

OPENOCD_BASE := ../../programmer/openocd/sparrowhawk/

# name of the project as defined in project file
PROJECT = project

JUNK = ${IMPL_DIR} .recovery ._Real_._Math_.vhd *.sty reportview.xml
JUNK += dummy_sym.sort project_tcl.html promote.xml
JUNK += generate_core.tcl generate_ngd.tcl msg_file.log
JUNK += project_tcr.dir

all: $(PROJECT)/$(PROJECT)_$(PROJECT).bit $(PROJECT)/$(PROJECT)_$(PROJECT)_sram.svf

$(PROJECT)/$(PROJECT)_$(PROJECT).bit:
	echo prj_project open ${PROJ_FILE} \; prj_run Export -task Bitgen | ${DIAMONDC}

$(PROJECT)/$(PROJECT)_$(PROJECT)_sram.vme: $(PROJECT)/$(PROJECT)_$(PROJECT).bit
	ddtcmd -oft -fullvme -if sparrowhawk_sram.xcf -nocompress -noheader -of $@

$(PROJECT)/$(PROJECT)_$(PROJECT)_sram.svf: $(PROJECT)/$(PROJECT)_$(PROJECT).bit
	ddtcmd -oft -svfsingle -revd -if sparrowhawk_sram.xcf -of $@
	# patch it to work with openocd
	patch -p0 < $(OPENOCD_BASE)/svf_patch.diff

program: $(PROJECT)/$(PROJECT)_$(PROJECT).bit
	echo pgr_project open sparrowhawk_sram.xcf \; pgr_program run | ${DIAMONDC}

program_wifi: $(PROJECT)/$(PROJECT)_$(PROJECT)_sram.svf
	openocd --file=$(OPENOCD_BASE)/remote.ocd --file=$(OPENOCD_BASE)/ecp3.ocd

program_ft2232: $(PROJECT)/$(PROJECT)_$(PROJECT)_sram.svf
	openocd --file=$(OPENOCD_BASE)/ft2232-fpu1.ocd --file=$(OPENOCD_BASE)/ecp3.ocd

#$(PROJECT)/$(PROJECT)_$(PROJECT).jed:
#	echo prj_project open ${PROJ_FILE} \; prj_run Export -task Jedecgen | ${DIAMONDC}

#$(PROJECT)/$(PROJECT)_$(PROJECT)_flash.vme: $(PROJECT)/$(PROJECT)_$(PROJECT).jed
#	ddtcmd -oft -fullvme -if sparrowhawk_flash.xcf -nocompress -noheader -of $@

#$(PROJECT)/$(PROJECT)_$(PROJECT)_flash.svf: $(PROJECT)/$(PROJECT)_$(PROJECT).jed
#	ddtcmd -oft -svfsingle -op "FLASH Erase,Program,Verify" -if $< -of $@

flash: $(PROJECT)/$(PROJECT)_$(PROJECT)_flash.vme
	${PROGRAMMERC} $<
	# after this, to gain access to serial port on linux
	# rmmod ftdi_sio; modprobe ftdi_sio

# example another project
#%.svf : %.jed
#	ddtcmd -oft -svfsingle -op "FLASH Erase,Program,Verify" -if $< -of $@
#	mv -f $@ $@.flash
#	ddtcmd -oft -svfsingle -revd -op "SRAM Fast Program" -if $< -of $@
#	mv -f $@ $@.sram
#	./svf_to_urjtag.pl <$@.flash | sed 's/,/./g' > $@

clean:
	rm -rf $(JUNK) *~
