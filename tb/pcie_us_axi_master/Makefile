# Copyright (c) 2020 Alex Forencich
#
# Permission is hereby granted, free of charge, to any person obtaining a copy
# of this software and associated documentation files (the "Software"), to deal
# in the Software without restriction, including without limitation the rights
# to use, copy, modify, merge, publish, distribute, sublicense, and/or sell
# copies of the Software, and to permit persons to whom the Software is
# furnished to do so, subject to the following conditions:
#
# The above copyright notice and this permission notice shall be included in
# all copies or substantial portions of the Software.
#
# THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR
# IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY
# FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE
# AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER
# LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM,
# OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN
# THE SOFTWARE.

TOPLEVEL_LANG = verilog

SIM ?= icarus
WAVES ?= 0

COCOTB_HDL_TIMEUNIT = 1ns
COCOTB_HDL_TIMEPRECISION = 1ps
export COCOTB_RESOLVE_X ?= RANDOM

DUT      = pcie_us_axi_master
TOPLEVEL = $(DUT)
MODULE   = test_$(DUT)
VERILOG_SOURCES += ../../rtl/$(DUT).v
VERILOG_SOURCES += ../../rtl/$(DUT)_rd.v
VERILOG_SOURCES += ../../rtl/$(DUT)_wr.v
VERILOG_SOURCES += ../../rtl/pcie_us_axis_cq_demux.v
VERILOG_SOURCES += ../../rtl/pulse_merge.v

# module parameters
export PARAM_AXIS_PCIE_DATA_WIDTH ?= 64
export PARAM_AXIS_PCIE_KEEP_WIDTH ?= $(shell expr $(PARAM_AXIS_PCIE_DATA_WIDTH) / 32 )
export PARAM_AXIS_PCIE_CQ_USER_WIDTH ?= $(if $(filter-out 512,$(PARAM_AXIS_PCIE_DATA_WIDTH)),88,183)
export PARAM_AXIS_PCIE_CC_USER_WIDTH ?= $(if $(filter-out 512,$(PARAM_AXIS_PCIE_DATA_WIDTH)),33,81)
export PARAM_AXI_DATA_WIDTH ?= $(PARAM_AXIS_PCIE_DATA_WIDTH)
export PARAM_AXI_ADDR_WIDTH ?= 64
export PARAM_AXI_STRB_WIDTH ?= $(shell expr $(PARAM_AXI_DATA_WIDTH) / 8 )
export PARAM_AXI_ID_WIDTH ?= 8
export PARAM_AXI_MAX_BURST_LEN ?= 256

SIM_BUILD ?= sim_build_$(MODULE)-$(PARAM_AXIS_PCIE_DATA_WIDTH)

ifeq ($(SIM), icarus)
	PLUSARGS += -fst

	COMPILE_ARGS += -P $(TOPLEVEL).AXIS_PCIE_DATA_WIDTH=$(PARAM_AXIS_PCIE_DATA_WIDTH)
	COMPILE_ARGS += -P $(TOPLEVEL).AXIS_PCIE_KEEP_WIDTH=$(PARAM_AXIS_PCIE_KEEP_WIDTH)
	COMPILE_ARGS += -P $(TOPLEVEL).AXIS_PCIE_CQ_USER_WIDTH=$(PARAM_AXIS_PCIE_CQ_USER_WIDTH)
	COMPILE_ARGS += -P $(TOPLEVEL).AXIS_PCIE_CC_USER_WIDTH=$(PARAM_AXIS_PCIE_CC_USER_WIDTH)
	COMPILE_ARGS += -P $(TOPLEVEL).AXI_DATA_WIDTH=$(PARAM_AXI_DATA_WIDTH)
	COMPILE_ARGS += -P $(TOPLEVEL).AXI_ADDR_WIDTH=$(PARAM_AXI_ADDR_WIDTH)
	COMPILE_ARGS += -P $(TOPLEVEL).AXI_STRB_WIDTH=$(PARAM_AXI_STRB_WIDTH)
	COMPILE_ARGS += -P $(TOPLEVEL).AXI_ID_WIDTH=$(PARAM_AXI_ID_WIDTH)
	COMPILE_ARGS += -P $(TOPLEVEL).AXI_MAX_BURST_LEN=$(PARAM_AXI_MAX_BURST_LEN)

	ifeq ($(WAVES), 1)
		VERILOG_SOURCES += iverilog_dump.v
		COMPILE_ARGS += -s iverilog_dump
	endif
else ifeq ($(SIM), verilator)
	COMPILE_ARGS += -Wno-SELRANGE -Wno-WIDTH -Wno-CASEINCOMPLETE -Wno-UNOPT

	COMPILE_ARGS += -GAXIS_PCIE_DATA_WIDTH=$(PARAM_AXIS_PCIE_DATA_WIDTH)
	COMPILE_ARGS += -GAXIS_PCIE_KEEP_WIDTH=$(PARAM_AXIS_PCIE_KEEP_WIDTH)
	COMPILE_ARGS += -GAXIS_PCIE_CQ_USER_WIDTH=$(PARAM_AXIS_PCIE_CQ_USER_WIDTH)
	COMPILE_ARGS += -GAXIS_PCIE_CC_USER_WIDTH=$(PARAM_AXIS_PCIE_CC_USER_WIDTH)
	COMPILE_ARGS += -GAXI_DATA_WIDTH=$(PARAM_AXI_DATA_WIDTH)
	COMPILE_ARGS += -GAXI_ADDR_WIDTH=$(PARAM_AXI_ADDR_WIDTH)
	COMPILE_ARGS += -GAXI_STRB_WIDTH=$(PARAM_AXI_STRB_WIDTH)
	COMPILE_ARGS += -GAXI_ID_WIDTH=$(PARAM_AXI_ID_WIDTH)
	COMPILE_ARGS += -GAXI_MAX_BURST_LEN=$(PARAM_AXI_MAX_BURST_LEN)

	ifeq ($(WAVES), 1)
		COMPILE_ARGS += --trace-fst
	endif
endif

iverilog_dump.v:
	echo 'module iverilog_dump();' > $@
	echo 'initial begin' >> $@
	echo '    $$dumpfile("$(TOPLEVEL).fst");' >> $@
	echo '    $$dumpvars(0, $(TOPLEVEL));' >> $@
	echo 'end' >> $@
	echo 'endmodule' >> $@

clean::
	@rm -rf sim_build_*
	@rm -rf iverilog_dump.v
	@rm -rf dump.fst $(TOPLEVEL).fst

include $(shell cocotb-config --makefiles)/Makefile.sim

