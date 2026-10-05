# AVR Assembly and the ATmega328P

Assembly is human-readable machine code. Assembly is one-to-one with machine code, meaning one assembly instruction maps to exactly one binary instruction the CPU executes. Every CPU has its own assembly language because every CPU has its own binary instruction format.

Instructions in AVR Assembly on the Arduino Uno R3 (with the ATmega328P microcontroller) follow this format:&#x20;

```
OPERATION  DESTINATION, SOURCE
```

The ATmega328P has 32 general-purpose **registers** named `R0` to `R31`, each one byte large. Registers are essentially ultrafast variables.

There are also **Immediate Values**, which are just literal numbers written directly into the instruction. For example, `0x05`. Hexadecimal is most commonly used as it maps cleanly to binary and is more readable.

The ATmega328P uses a modified version of the Harvard architecture, which has three separate memory spaces:

<table data-search="false"><thead><tr><th width="241">Memory Space</th><th width="420">What it does</th><th width="592">How to access in assembly</th></tr></thead><tbody><tr><td>Program Memory (Flash)</td><td>Stores your program code</td><td>You typically will not write code that changes the Flash, but <code>LPM</code> in assembly is used to <em><strong>L</strong></em>oad <em><strong>P</strong></em>rogram <em><strong>M</strong></em>emory.</td></tr><tr><td>Data Memory</td><td>Data memory is split up into four sections:</td><td></td></tr><tr><td>      <strong>⊢</strong> <em>32 Registers</em></td><td><ul><li><em>As mentioned above, essentially ultrafast variables</em></li></ul></td><td><strong>Writing:</strong><br>- Use <code>LDI</code> to write a number into registers (<code>R16</code> to <code>R31</code> ONLY)<br>- Use <code>MOV</code> to copy the value from one register to another<br>- Use <code>LDS</code> to load a value from a data memory address<br><strong>Reading:</strong><br>- Use <code>STS</code> to write a register's contents into a data memory address<br>- Use the register in an operation like <code>ADD R2, R14</code></td></tr><tr><td>      <strong>⊢</strong> <em>64 I/O Registers</em></td><td><ul><li><em>Controls the digital input/output pins. Connects software code to physical hardware pins</em></li></ul></td><td><strong>Writing:</strong><br>- Use <code>IN</code> to read data from an I/O register into a general working register<br><strong>Reading:</strong><br>- Use <code>OUT</code> to write data from a general working register into an I/O register</td></tr><tr><td>      <strong>⊢</strong> <em>160 Ext. I/O Registers</em></td><td><ul><li>Extended I/O Registers are essentially just more I/O registers in a different spot in the Data Memory</li></ul></td><td>You cannot use <code>IN</code> and <code>OUT</code> for extended I/O registers, you must use <code>LDS</code> and <code>STS</code> as seen above.</td></tr><tr><td>      <strong>⊢</strong> <em>Internal SRAM</em></td><td><ul><li>The rest of the Data Memory, a "scratchpad" to temporarily store stuff</li></ul></td><td>You can use <code>LDS</code> and <code>STS</code> again, or use <code>ST</code> and <code>LD</code> with a <a data-footnote-ref href="#user-content-fn-1">pointer register</a></td></tr><tr><td>EEPROM Memory</td><td>Non-volatile (permanent) storage that holds stuff like user configurations, calibration data, and variables. Much smaller than Flash memory but can be changed byte-by-byte instead of by entire sectors</td><td>Reading and writing EEPROM memory is more complex due to changes being irreversible and safety features that prevent the corruption of the EEPROM. More information on it below.</td></tr></tbody></table>

### Reading and Writing to the EEPROM

#### Important Registers and Bits

* `EEARH` and `EEARL`: EEPROM [Address Register (High/Low bytes)](#user-content-fn-2)[^2]
* `EEDR`: EEPROM Data Register (Holds the byte to write)
* `EECR`: EEPROM Control Register. It is broken into eight bits, three of which we will need for our read/write operations
  * Bit 0 holds the `EERE`, known as the **EEPROM Read Enable**. Setting it to `1` will trigger a read operation, which is saved to the `EEDR` register (seen above).
  * Bit 1 holds the `EEPE`, known as the **EEPROM Write Enable**. Setting it to `1` will start writing to EEPROM, and when it is `0` it signifies that there is no current write operation.
  * Bit 2 holds the `EEMPE`, known as the **EEPROM Master Write Enable**. This bit must be enabled to start preparation to write to the EEPROM.

#### Writing to EEPROM

1. Wait for EEPE to equal 0
2. Set up address registers by writing to EEARH and EEARL
3. Save the status register SREG and disable global interrupts using cli
4. Set EEMPE to 1, allowing the EEPE to get modified
5. Start the write by setting EEPE to 1, and finally restore the status register we saved earlier.

#### Reading the EEPROM

1. Wait for EEPE to equal 0
2. Set up address registers by writing to EEARH and EEARL
3. Start the read by setting EERE to 1
4. Read the data in the EEDR register

[^1]: The ATmega328P has three pointer pairs that point to locations in memory: X, Y, and Z. X is made from `R26:R27`, Y from `R28:R29`, and Z from `R30:R31`.

[^2]: Defines the address you want to Read/Write. This address is broken into two parts, hence why there are two registers. `EEARH` stores the most significant bits, whilst `EEARL` stores the least significant bits. For example, address `0x0105` would need `EEARH` to store `0x01` and `EEARL` to store `0x05`.
