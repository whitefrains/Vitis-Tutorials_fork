.PHONY: help

help::
	@echo  " Makefile Usage:"
	@echo  ""
	@echo  "  make build TARGET=<sw_emu/hw_emu/hw> STEP=<baseline/localbuf/fixedpoint/dataflow/multicu> SOLUTION=1"
	@echo  "  Command to generate the design for specified target and step"
	@echo  ""
	@echo  "  make run TARGET=<sw_emu/hw_emu/hw> STEP=<baseline/localbuf/fixedpoint/dataflow/multicu> SOLUTION=1"
	@echo  "  Command to generate,run and verifiy the design for specified target and step"
	@echo  ""
	@echo  "  make clean TARGET=<sw_emu/hw_emu/hw> STEP=<baseline/localbuf/fixedpoint/dataflow/multicu>"
	@echo  "  Command to remove the generated files for specified target and step."
	@echo  ""
	@echo  "  make view_prof_report TARGET=<sw_emu/hw_emu/hw> STEP=<baseline/localbuf/fixedpoint/dataflow/multicu>"
	@echo  "  Command to view profile summary report in vitis_analyzer utility"
	@echo  ""
	@echo  "  make view_timeline_trace TARGET=<sw_emu/hw_emu/hw> STEP=<baseline/localbuf/fixedpoint/dataflow/multicu>"
	@echo  "  Command to view application timeline report in vitis_analyzer utility"
	@echo  ""


## platform selection
PLATFORM := xilinx_u200_xdma_201830_2

## TARGET can be set as:
## sw_emu: software emulation
## hw_emu: hardware Emulation
## hw: hardware run
TARGET := hw_emu

## Below are the names for host executable and xclbin.
## Please keep it unchanged.
HOST_EXE := convolution.exe
XO_NAME := convolve_fpga_$(TARGET)
XCLBIN := fpga_container_$(TARGET).xclbin



## Source file repository selection
## Step1           - baseline  (The baseline version design)
## Step2           - localbuf  (Use local buffer to store the data for processing)
## Step3           - fixedpoint (Use ap_fixed type data)
## Step4           - dataflow  (Adopt dataflow and static loops optimizations)
## Step5           - multicu (Adopt different optimizations on host code side, such as out-of-order command queue and multiple compute units)

## BUILD_DIR are used for the generated files, including .exe, .xclbin, and the reports/logs. BUILD_DIR selection should be matched to SRC_REPO for better structure.
PROJECT_DIR := ..

## Below listed the different steps of the design.
## The descriptions are in 'HowToRunTutorial.md'.
## YOu can change the STEP here or specify it when launching makefile.

STEP  := baseline

ifeq ($(SOLUTION), 1)
SRC_REPO := $(PROJECT_DIR)/reference-files/$(STEP)
else
SRC_REPO := $(PROJECT_DIR)/src/$(STEP)
endif

BUILD_DIR := $(PROJECT_DIR)/build/$(STEP)


## Kernel Parameters

## Compute Unit Number
## Set to 1 for STEP=<baseline/localbuf/fixedpoint/dataflow>
## Set to 4 for STEP=<multicu>

ifeq ($(STEP), multicu)
CU_NUM := 4
else
CU_NUM := 1
endif

## Host Application files repository

HOST_SRC_CPP := $(SRC_REPO)/common.cpp
HOST_SRC_CPP += $(SRC_REPO)/convolve_kernel.cpp
HOST_SRC_CPP += $(SRC_REPO)/grayscale_kernel.cpp
HOST_SRC_CPP += $(SRC_REPO)/xcl2.cpp
HOST_SRC_CPP += $(SRC_REPO)/convolve.cpp
HOST_SRC_CPP += $(SRC_REPO)/main.cpp
HOST_SRC_H := $(SRC_REPO)/common.h
HOST_SRC_H += $(SRC_REPO)/kernels.h
HOST_SRC_H += $(SRC_REPO)/xcl2.hpp
HOST_SRC_H += $(SRC_REPO)/constants.h



## Kernel Source Files repository

KERNEL_SRC_CPP := $(SRC_REPO)/convolve_fpga.cpp
KERNEL_SRC_H := $(SRC_REPO)/constants.h
KERNEL_SRC_H += $(SRC_REPO)/kernels.h
KERNEL_SRC_H += $(SRC_REPO)/types.h
KERNEL_SRC_H_DIR := $(SRC_REPO)


# Host Compiler Global Settings and Include Libraries

#CXXFLAGS :=
CXXFLAGS += -D__USE_XOPEN2K8
CXXFLAGS += -D__USE_XOPEN2K8
CXXFLAGS += -I$(XILINX_XRT)/include/
CXXFLAGS += -I$(XILINX_VIVADO)/include/
CXXFLAGS += -I$(SRC_REPO)
CXXFLAGS += -O0 -g -Wall -fmessage-length=0 -std=c++11

## The following will enable the available solution code for each step
ifeq ($(SOLUTION), 1)
CXXFLAGS += -DBASELINE_ADD -DHOST_CODE_OPT -DLOCAL_BUF_OPT -DDF_OPT -DFP_OPT
endif


## Default mode sets 1920x1080 input video format for all types of runs
## Set # of frame to 1 and scale to 512x10 for quicker turnaround of development cycle
ifeq ($(NUM_FRAMES), 1)
ifeq ($(STEP), multicu)
RUN_OPTS = --scale 512 40 --nframes 1
GOLDEN_OUT = golden_out_small_40.mp4
else
RUN_OPTS = --scale 512 10 --nframes 1
GOLDEN_OUT = golden_out_small.mp4
endif
else
GOLDEN_OUT = golden_out_full.mp4
endif

CXXLDFLAGS := -L$(XILINX_XRT)/lib/
CXXLDFLAGS += -lxilinxopencl -lpthread -lrt


## Kernel Compiler and Linker Flags

VPPFLAGS := -t $(TARGET)
VPPFLAGS += --config design.cfg
VPPFLAGS += -I$(KERNEL_SRC_H_DIR)
VPPFLAGS += --temp_dir $(BUILD_DIR)
VPPFLAGS += --log_dir $(BUILD_DIR)

ifeq ($(SOLUTION), 1)
VPPFLAGS += -DBASELINE_ADD -DHOST_CODE_OPT -DLOCAL_BUF_OPT -DDF_OPT -DFP_OPT
endif




## Host Executable File Generation

$(BUILD_DIR)/$(HOST_EXE): $(HOST_SRC_CPP) $(HOST_SRC_H)
	mkdir -p $(BUILD_DIR)
	g++ $(CXXFLAGS) $(HOST_SRC_CPP) $(CXXLDFLAGS) -o $@


## Kernel XO and Xclbin File Generation

$(BUILD_DIR)/$(XO_NAME).xo: $(KERNEL_SRC_CPP) $(KERNEL_SRC_H)
	mkdir -p $(BUILD_DIR)
	v++ $(VPPFLAGS) -c -k convolve_fpga $(KERNEL_SRC_CPP) -o $@

$(BUILD_DIR)/$(XCLBIN): $(BUILD_DIR)/$(XO_NAME).xo
	mkdir -p $(BUILD_DIR)
	v++ $(VPPFLAGS) -l -o $@ $(BUILD_DIR)/$(XO_NAME).xo

## Emulation Files Generation

EMCONFIG_FILE = emconfig.json

$(BUILD_DIR)/$(EMCONFIG_FILE):
	 emconfigutil --nd 1  --platform $(PLATFORM) --od $(BUILD_DIR)


#
# primary build targets
#

.PHONY: all clean

## build the design without running host application

build: $(BUILD_DIR)/$(HOST_EXE) $(BUILD_DIR)/$(XCLBIN) $(BUILD_DIR)/$(EMCONFIG_FILE)


## build the design and then run host application
## run with 512x10 input video format for emulation and 1280x720 for hardware

run: build
	cp xrt.ini $(BUILD_DIR);
ifeq ($(TARGET), hw)
	cd $(BUILD_DIR) && unset XCL_EMULATION_MODE;    ./$(HOST_EXE)  --kernel_name convolve_fpga $(RUN_OPTS) ../../video.mp4 ./$(XCLBIN) ;
else
	cd $(BUILD_DIR) && XCL_EMULATION_MODE=$(TARGET) ./$(HOST_EXE)  --kernel_name convolve_fpga $(RUN_OPTS) ../../video.mp4 ./$(XCLBIN) ;
endif
	cd $(BUILD_DIR) && mv ./profile_summary.csv sdaccel_profile_summary_$(TARGET).csv;
	cd $(BUILD_DIR) && mv ./timeline_trace.csv sdaccel_timeline_trace_$(TARGET).csv;
## compare the output result with golden result
	cmp $(BUILD_DIR)/output.mp4 ../$(GOLDEN_OUT); RETVAL=$$?; \
	if [ $$RETVAL -eq \0 ];  then \
	  echo "\n\nPASS : Output Video Matches the Golden Output Results \n\n\n" ; \
	else \
	  echo "\n\nFAIL : Output Video Corrupt\n\n\n" ; \
	fi



## view profile summary report in Vitis Analyzer GUI

view_prof_report:
	cd $(BUILD_DIR) && vitis_analyzer -open ./sdaccel_profile_summary_$(TARGET).csv


## view timeline trace report in Vitis Analyzer GUI

view_timeline_trace:
	cd $(BUILD_DIR) && vitis_analyzer -open ./sdaccel_timeline_trace_$(TARGET).csv

## Clean generated files

clean:
	rm -rf $(BUILD_DIR)/$(XCLBIN) $(BUILD_DIR)/$(HOST_EXE) $(BUILD_DIR)/$(EMCONFIG_FILE) $(BUILD_DIR)/$(XO_NAME).xo $(BUILD_DIR)/*.ltx $(BUILD_DIR)/*_$(TARGET).log $(BUILD_DIR)/v++_*_$(TARGET)_* $(BUILD_DIR)/_x* $(BUILD_DIR)/*.info $(BUILD_DIR)/convolve_fpga_$(TARGET)* $(BUILD_DIR)/link $(BUILD_DIR)/reports/convolve_fpga_$(TARGET)
