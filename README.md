# Xilinx Single-Port Block RAM (VHDL)

[![Platform](https://img.shields.io/badge/FPGA-Xilinx-red)](https://www.xilinx.com/)
[![Language](https://img.shields.io/badge/HDL-VHDL-green)](https://en.wikipedia.org/wiki/VHDL)
[![Memory Type](https://img.shields.io/badge/Type-Block%20RAM-blue)](https://www.xilinx.com/products/intellectual-property/block_memory_generator.html)
[![License](https://img.shields.io/badge/License-MIT-yellow.svg)](LICENSE)

A parameterizable, synthesizable **single-port Block RAM** implementation in VHDL targeting Xilinx FPGAs. Features configurable data width, depth, and performance modes with read-first semantics.

## Table of Contents
- [Overview](#overview)
- [Features](#features)
- [Architecture](#architecture)
- [Port Description](#port-description)
- [Generic Parameters](#generic-parameters)
- [Usage Examples](#usage-examples)
- [Performance Modes](#performance-modes)
- [Technical Details](#technical-details)
- [Synthesis](#synthesis)
- [Author](#author)

---

## Overview

This module implements a **single-port synchronous RAM** with read-first behavior, specifically optimized for Xilinx FPGA Block RAM (BRAM) primitives. The design is fully parameterizable and suitable for use in:

- **Processor memory subsystems** (instruction memory, data memory)
- **FIFO buffers** and circular buffers
- **Lookup tables** (LUTs) with large datasets
- **Data caching** and temporary storage
- **Educational projects** demonstrating BRAM usage

**Key Characteristics:**
- Single read/write port with synchronous operation
- Configurable data width (default: 16 bits)
- Configurable depth (default: 128 words)
- Automatic address bus width calculation
- Two performance modes: LOW_LATENCY and HIGH_PERFORMANCE
- Read-first semantics (simultaneous read/write returns old data)

---

## Features

### Core Capabilities
- **Parameterized Design**: Configurable width and depth via VHDL generics
- **Automatic Address Calculation**: Dynamic address bus width using `log2()` function
- **Dual Performance Modes**: Choose between 1-cycle or 2-cycle read latency
- **Block RAM Targeting**: Uses Xilinx `ram_style` attribute for BRAM inference
- **Read-First Behavior**: Predictable simultaneous read/write semantics
- **Synchronous Operation**: Single clock domain, rising-edge triggered

### Design Philosophy
- **Simplicity**: Clean, readable VHDL with minimal complexity
- **Portability**: Standard VHDL with Xilinx-specific attributes isolated
- **Reusability**: Generic interface suitable for various applications
- **Efficiency**: Direct BRAM mapping for optimal FPGA resource usage

---

## Architecture

### Block Diagram

```
                    ┌─────────────────────────────────────┐
                    │  xilinx_single_port_ram_read_first │
                    └────────────┬────────────────────────┘
                                 │
         ┌───────────────────────┼────────────────────────┐
         │                       │                        │
         ▼                       ▼                        ▼
    ┌─────────┐          ┌─────────────┐         ┌──────────┐
    │ Address │          │ Write Data  │         │  Clock   │
    │  Input  │          │    Input    │         │  Enable  │
    └────┬────┘          └──────┬──────┘         └─────┬────┘
         │                      │                       │
         └──────────────────────┼───────────────────────┘
                                │
                                ▼
                    ┌────────────────────────┐
                    │   BRAM Storage Array   │
                    │   (RAM_DEPTH words)    │
                    │   × (RAM_WIDTH bits)   │
                    └────────────┬───────────┘
                                 │
                    ┌────────────▼───────────┐
                    │   Performance Mode     │
                    │  LOW_LATENCY: Direct   │
                    │ HIGH_PERF: Registered  │
                    └────────────┬───────────┘
                                 │
                                 ▼
                         ┌───────────────┐
                         │  Read Data    │
                         │    Output     │
                         └───────────────┘
```

### Memory Organization

```
Address 0   ┌──────────────────┐
            │  Word 0 (16-bit) │
Address 1   ├──────────────────┤
            │  Word 1 (16-bit) │
Address 2   ├──────────────────┤
            │  Word 2 (16-bit) │
    ...     │       ...        │
Address 127 ├──────────────────┤
            │ Word 127 (16-bit)│
            └──────────────────┘

Default Configuration:
  - 128 words deep
  - 16 bits wide
  - 7-bit address bus (log2(128) = 7)
  - Total capacity: 2048 bits (256 bytes)
```

---

## Port Description

### Entity Declaration

```vhdl
entity xilinx_single_port_ram_read_first is
    generic (
        RAM_WIDTH       : integer := 16;              -- Data width in bits
        RAM_DEPTH       : integer := 128;             -- Number of words
        RAM_PERFORMANCE : string  := "LOW_LATENCY";   -- Performance mode
        INIT_FILE       : string  := "RAM_INIT.dat"   -- Initialization file
    );
    port (
        address_i         : in  std_logic_vector((log2(RAM_DEPTH)-1) downto 0);
        ram_input_data_i  : in  std_logic_vector(RAM_WIDTH-1 downto 0);
        clk_i             : in  std_logic;
        write_enable_i    : in  std_logic;
        ram_output_data_o : out std_logic_vector(RAM_WIDTH-1 downto 0)
    );
end entity;
```

### Port Signals

| Port Name | Direction | Width | Description |
|-----------|-----------|-------|-------------|
| **address_i** | Input | log2(RAM_DEPTH) | Memory address (word-addressed) |
| **ram_input_data_i** | Input | RAM_WIDTH | Write data input |
| **clk_i** | Input | 1 | Clock signal (rising edge active) |
| **write_enable_i** | Input | 1 | Write enable (1 = write, 0 = read-only) |
| **ram_output_data_o** | Output | RAM_WIDTH | Read data output |

**Address Bus Width Calculation:**
- Automatically calculated using `log2()` function
- Examples:
  - RAM_DEPTH = 128 → address width = 7 bits
  - RAM_DEPTH = 256 → address width = 8 bits
  - RAM_DEPTH = 1024 → address width = 10 bits

---

## Generic Parameters

### Configuration Options

| Generic | Type | Default | Description | Valid Range |
|---------|------|---------|-------------|-------------|
| **RAM_WIDTH** | integer | 16 | Data bus width in bits | 1 to 1024+ |
| **RAM_DEPTH** | integer | 128 | Number of memory locations | 2 to 65536+ |
| **RAM_PERFORMANCE** | string | "LOW_LATENCY" | Performance mode | "LOW_LATENCY" or "HIGH_PERFORMANCE" |
| **INIT_FILE** | string | "RAM_INIT.dat" | Initialization file path | Any valid path |

### Examples

**Small 8-bit RAM (64 words):**
```vhdl
generic map (
    RAM_WIDTH => 8,
    RAM_DEPTH => 64,
    RAM_PERFORMANCE => "LOW_LATENCY"
)
```

**Large 32-bit RAM (4096 words):**
```vhdl
generic map (
    RAM_WIDTH => 32,
    RAM_DEPTH => 4096,
    RAM_PERFORMANCE => "HIGH_PERFORMANCE"
)
```

**Custom Configuration:**
```vhdl
generic map (
    RAM_WIDTH => 24,
    RAM_DEPTH => 512,
    RAM_PERFORMANCE => "LOW_LATENCY",
    INIT_FILE => "my_init_data.dat"
)
```

---

## Usage Examples

### Basic Instantiation

```vhdl
library ieee;
use ieee.std_logic_1164.all;

entity my_design is
    port (
        clk   : in  std_logic;
        addr  : in  std_logic_vector(6 downto 0);
        din   : in  std_logic_vector(15 downto 0);
        we    : in  std_logic;
        dout  : out std_logic_vector(15 downto 0)
    );
end entity;

architecture rtl of my_design is
begin

    -- Instantiate RAM with default parameters
    ram_inst : entity work.xilinx_single_port_ram_read_first
    port map (
        address_i         => addr,
        ram_input_data_i  => din,
        clk_i             => clk,
        write_enable_i    => we,
        ram_output_data_o => dout
    );

end architecture;
```

### Custom Configuration Example

```vhdl
architecture rtl of my_processor is
    signal mem_addr : std_logic_vector(9 downto 0);  -- 10-bit address
    signal mem_din  : std_logic_vector(31 downto 0); -- 32-bit data
    signal mem_dout : std_logic_vector(31 downto 0);
    signal mem_we   : std_logic;
begin

    -- Instantiate 1K × 32-bit data memory
    data_memory : entity work.xilinx_single_port_ram_read_first
    generic map (
        RAM_WIDTH       => 32,
        RAM_DEPTH       => 1024,
        RAM_PERFORMANCE => "HIGH_PERFORMANCE",
        INIT_FILE       => ""  -- No initialization
    )
    port map (
        address_i         => mem_addr,
        ram_input_data_i  => mem_din,
        clk_i             => clk,
        write_enable_i    => mem_we,
        ram_output_data_o => mem_dout
    );

end architecture;
```

---

## Performance Modes

### LOW_LATENCY Mode

**Characteristics:**
- **Read Latency**: 1 clock cycle
- **Output**: Combinational path from BRAM to output port
- **Timing**: Longer clock-to-output delay
- **Use Case**: When read latency is critical

**Timing Diagram:**
```
Clock:    ___┌───┐___┌───┐___┌───┐___
Address:  ─< A0 >───< A1 >───< A2 >───
WE:       ────────┌─────────┐─────────
Din:      ────────< D1 >─────────────
Dout:     ─< [A0] >───< [A1] >───< D1 >
          └─ 1 cycle ─┘
```

**Implementation:**
```vhdl
-- Direct assignment (no output register)
no_output_register : if C_RAM_PERFORMANCE = "LOW_LATENCY" generate
    ram_output_data_o <= ram_data;
end generate;
```

### HIGH_PERFORMANCE Mode

**Characteristics:**
- **Read Latency**: 2 clock cycles
- **Output**: Registered output for better timing
- **Timing**: Improved clock-to-output performance
- **Use Case**: When clock speed is critical

**Timing Diagram:**
```
Clock:    ___┌───┐___┌───┐___┌───┐___┌───┐
Address:  ─< A0 >───< A1 >───< A2 >───────
WE:       ────────┌─────────┐─────────────
Din:      ────────< D1 >─────────────────
Dout:     ─────────────< [A0] >───< [A1] >
          └──── 2 cycles ────┘
```

**Implementation:**
```vhdl
-- Output register for timing optimization
output_register : if C_RAM_PERFORMANCE = "HIGH_PERFORMANCE" generate
    process(clk_i) begin
        if(rising_edge(clk_i)) then
            ram_output_data_reg <= ram_data;
        end if;
    end process;
    ram_output_data_o <= ram_output_data_reg;
end generate;
```

### Performance Comparison

| Metric | LOW_LATENCY | HIGH_PERFORMANCE |
|--------|-------------|------------------|
| **Read Latency** | 1 cycle | 2 cycles |
| **Clock-to-Output** | Longer | Shorter |
| **Max Frequency** | Lower | Higher |
| **Pipeline Stages** | 0 (combinational) | 1 (registered) |
| **Best For** | Latency-sensitive | Frequency-critical |

---

## Technical Details

### Read-First Semantics

**Simultaneous Read/Write Behavior:**
When reading and writing to the **same address** in the **same clock cycle**, the RAM returns the **old data** (before write).

```vhdl
process(clk_i) begin
    if(rising_edge(clk_i)) then
        if(write_enable_i = '1') then
            BRAM_virtual(to_integer(unsigned(address_i))) <= ram_input_data_i;
        end if;

        -- Read happens BEFORE write is visible
        ram_data <= BRAM_virtual(to_integer(unsigned(address_i)));
    end if;
end process;
```

**Example:**
```
Cycle 1: Write 0xABCD to address 0x10 (old value = 0x0000)
         Read from address 0x10 → Returns 0x0000 (old value)

Cycle 2: Read from address 0x10 → Returns 0xABCD (updated value)
```

### Memory Data Type

**2D Array Declaration:**
```vhdl
type T_RAM is array (C_RAM_DEPTH-1 downto 0)
    of std_logic_vector(C_RAM_WIDTH-1 downto 0);

signal BRAM_virtual : T_RAM := (others => (others => '0'));
```

**Initialization:**
- All locations initialized to zero: `(others => (others => '0'))`
- Each word: `std_logic_vector(RAM_WIDTH-1 downto 0)`
- Array indices: 0 to RAM_DEPTH-1

### Xilinx BRAM Targeting

**Synthesis Attribute:**
```vhdl
attribute ram_style : string;
attribute ram_style of BRAM_virtual : signal is "block";
```

**Effect:**
- Forces Xilinx synthesizer to map `BRAM_virtual` to Block RAM primitives
- Without this attribute, synthesizer might use distributed RAM (LUTs)
- Ensures efficient FPGA resource usage

**Alternative Attributes:**
```vhdl
-- Use distributed RAM (LUT-based)
attribute ram_style of BRAM_virtual : signal is "distributed";

-- Let synthesizer decide
attribute ram_style of BRAM_virtual : signal is "auto";
```

### Utility Package (ram_pkg)

**Log2 Function:**
```vhdl
function log2(depth : natural) return integer is
    variable temp    : integer := depth;
    variable ret_val : integer := 0;
begin
    while temp > 1 loop
        ret_val := ret_val + 1;
        temp    := temp / 2;
    end loop;
    return ret_val;
end function;
```

**Purpose**: Calculates ceiling(log2(depth)) for address bus width

**Examples:**
- `log2(128) = 7` (128 requires 7 bits: 0-127)
- `log2(256) = 8` (256 requires 8 bits: 0-255)
- `log2(1024) = 10` (1024 requires 10 bits: 0-1023)

---

## Synthesis

### Xilinx Vivado Synthesis

**Expected Resource Usage** (Xilinx 7-series):

| Configuration | Block RAMs | LUTs | FFs |
|---------------|------------|------|-----|
| 128×16 (default) | 1 | ~10 | ~20 |
| 512×32 | 1 | ~15 | ~35 |
| 1024×32 | 2 | ~15 | ~35 |
| 2048×64 | 4 | ~20 | ~70 |

**Note**: Actual usage depends on FPGA family, synthesis settings, and performance mode.

### Synthesis Recommendations

**For Maximum Frequency:**
```vhdl
generic map (
    RAM_PERFORMANCE => "HIGH_PERFORMANCE"
)
```
- Use output register
- Better timing closure
- Enables higher clock speeds

**For Minimum Latency:**
```vhdl
generic map (
    RAM_PERFORMANCE => "LOW_LATENCY"
)
```
- Direct combinational output
- 1-cycle read latency
- May limit maximum frequency

### BRAM vs. Distributed RAM

| Feature | Block RAM | Distributed RAM (LUTs) |
|---------|-----------|------------------------|
| **Capacity** | Large (18Kb, 36Kb blocks) | Small (<512 bits) |
| **Speed** | Fast (dedicated routing) | Variable |
| **Resource** | Dedicated BRAM | Uses logic LUTs |
| **This Design** | ✓ Targets BRAM | Can be modified |

---

## Integration with RISC-V Processor

This RAM module can be used as the memory primitive for RISC-V processors:

### Instruction Memory (ROM-Style)

```vhdl
-- Read-only instruction memory
instruction_memory : entity work.xilinx_single_port_ram_read_first
generic map (
    RAM_WIDTH       => 32,
    RAM_DEPTH       => 1024,
    RAM_PERFORMANCE => "LOW_LATENCY",
    INIT_FILE       => "program.dat"
)
port map (
    address_i         => pc(11 downto 2),  -- Word-aligned PC
    ram_input_data_i  => (others => '0'),  -- No writes
    clk_i             => clk,
    write_enable_i    => '0',              -- Always read-only
    ram_output_data_o => instruction
);
```

### Data Memory (Read/Write)

```vhdl
-- Read-write data memory
data_memory : entity work.xilinx_single_port_ram_read_first
generic map (
    RAM_WIDTH       => 32,
    RAM_DEPTH       => 1024,
    RAM_PERFORMANCE => "HIGH_PERFORMANCE",
    INIT_FILE       => ""
)
port map (
    address_i         => data_addr,
    ram_input_data_i  => write_data,
    clk_i             => clk,
    write_enable_i    => mem_write,
    ram_output_data_o => read_data
);
```

---

## Limitations

### Current Implementation

- **Single Port**: Only one read or write per cycle
- **No Byte Enable**: Writes entire word (no byte-level access)
- **Synchronous Only**: No asynchronous read support
- **Word-Addressed**: Not byte-addressed
- **Xilinx-Specific**: Uses Xilinx synthesis attributes

### Potential Enhancements

Future improvements could include:
- Dual-port RAM (simultaneous read/write)
- Byte-enable signals for partial word writes
- True dual-port (two independent read/write ports)
- Asynchronous read option
- Byte-addressable interface with byte enables
- Generic BRAM targeting (portable across vendors)

---

## File Structure

```
RAM/
├── README.md              # This file
├── .gitignore            # Git ignore rules
└── RAM.srcs/
    └── sources_1/new/
        └── RAM.vhd       # Main implementation
            ├── ram_pkg package (log2 function)
            └── xilinx_single_port_ram_read_first entity
```

---

## License

This project is licensed under the **MIT License**.

```
Copyright (c) 2024 Mehmet Arslan

Permission is hereby granted, free of charge, to any person obtaining a copy
of this software and associated documentation files (the "Software"), to deal
in the Software without restriction...
```

---

## Author

**Mehmet Arslan**

Hardware design engineer specializing in FPGA development and memory subsystem design.

**Technical Skills Demonstrated:**
- VHDL RTL design
- Xilinx FPGA Block RAM targeting
- Parameterizable memory modules
- Synchronous digital design
- FPGA synthesis optimization
- Memory architecture design

**GitHub**: [@htmos6](https://github.com/htmos6) | [@marslan6](https://github.com/marslan6)

---

## References

### Xilinx Documentation
- [UG901: Vivado Synthesis User Guide](https://www.xilinx.com/support/documentation/sw_manuals/xilinx2020_1/ug901-vivado-synthesis.pdf)
- [UG473: 7 Series FPGAs Memory Resources](https://www.xilinx.com/support/documentation/user_guides/ug473_7Series_Memory_Resources.pdf)
- [XST RAM HDL Coding Techniques](https://www.xilinx.com/support/documentation/sw_manuals/xilinx11/xst.pdf)

### VHDL Learning
- [Free Range VHDL](https://github.com/fabriziotappero/Free-Range-VHDL-book)
- [VHDL Tutorial](http://www.vhdl-online.de/tutorial/)

---

**Last Updated**: January 2026
**Version**: 1.0
**Repository**: https://github.com/marslan6/RAM
