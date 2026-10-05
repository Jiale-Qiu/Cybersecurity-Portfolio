---
description: My notes whilst learning the Fetch-Decode-Execute cycle.
---

# Fetch-Decode-Execute Cycle

The Fetch-Decode-Execute (FDE) cycle is the process every CPU uses to run programs. It has three phases:

{% stepper %}
{% step %}
### Fetch

The CPU reads the next instruction from program memory and loads it into the **instruction register**. The **instruction register** holds the instruction that is currently being executed or decoded (see the next steps?). Almost all CPUs have an instruction register.

The CPU knows what the next instruction is by reading the **program counter**. The program counter is another special register that stores the address of the next instruction to be executed.

{% hint style="info" %}
Both the instruction register and program counter are registers. Registers are tiny storage locations that hold data currently being used by the CPU. Although they are tiny, they are the fastest form of memory in a computer, even beating cache memory and RAM!
{% endhint %}
{% endstep %}

{% step %}
### Decode

In the decode step, the Control Unit (the core part of the CPU; acts as an orchestrator) reads the instruction register and determines the next action the processor is supposed to perform. Additionally, it sets stuff up in the CPU to prepare for execution.

In the ATmega328P, most instructions are 16 bits long, although a few take 32 bits instead. The control unit always reads the first 16 bits of the instruction — no matter its size — and decides if it needs more bits based on what those first 16 bits contain.
{% endstep %}

{% step %}
### Execute

The final step of the FDE cycle — everything the decode step has prepared fires off and starts running.
{% endstep %}
{% endstepper %}

## Manually Tracing a Simple Program for the ATmega328P&#x20;

With knowledge of the FDE cycle, I'll see if I can manually trace this. For the sake of simplicity, we are assuming `LDI R16, 0x05` is the very first instruction the CPU runs on start.

```
; Initialize: R16 = 0x05, R17 = 0x03
; Then add them, store result in R17

LDI R16, 0x05      ; Load Immediate: R16 = 5
LDI R17, 0x03      ; Load Immediate: R17 = 3
ADD R17, R16       ; Add: R17 = R17 + R16; R17 = 8
```

{% stepper %}
{% step %}
### **Instruction 1: LDI R16, 0x05**

#### <mark style="color:blue;">**Fetch**</mark>

First, the CPU will **fetch** the instruction from the flash. The program counter starts at `0x0000`, so the CPU reads from address `0x0000` and saves the value to the instruction register. The program counter then increments by 1, bringing it up to `0x0001`.

#### <mark style="color:$success;">**Decode**</mark>

The CPU's control unit then reads from the instruction register and sees `1110 0000 0000 0101`. The control unit sees the `1110` at the very beginning, which lets it know that the opcode is `LDI` (Load Immediate). The `LDI` opcode follows this binary format: `1110 KKKK dddd KKKK` (We can reference the Instruction Set Manual [Here](https://ww1.microchip.com/downloads/en/DeviceDoc/AVR-InstructionSet-Manual-DS40002198.pdf))

* `KKKK ... KKKK` is the constant value you want to write into the register. It is broken up into two 4-bit sections. The&#x20;
* `dddd` is the destination register. Notice that the ATmega328P has 32 general-purpose working registers, but 4 bits can only store 16 different values. This is why you can only use registers `R15`-`R31` when using `LDI`. So `0000` equals `R16`, `0001` equals `R17`, `0010` equals `R18`, and so on.

Let's continue with decoding our binary, `1110 0000 0000 0101`. When we combine the `KKKK` values, we get `00000101`, which equals 5 in binary — just like `0x05` in our original instruction!

Decoding our `dddd` value, `0000` corresponds to `R16` — which exactly matches up with `R16` in our original instruction!

Now the control unit knows:

* The instruction is `LDI`&#x20;
* The instruction wants the constant `0x05` to be written into `R16` (register 16)

#### <mark style="color:$danger;">**Execute**</mark>**&#x20;+** <mark style="color:blue;">**Fetch**</mark>

Now that the Control Unit has finished setting everything up, it actually runs the operation. `0x05` gets saved into `R16`. Although the FDE (Fetch-Decode-Execute) cycle is taught as going one step at a time, many microcontrollers — like the ATmega328P — use pipelining. Instead of waiting for **execute** to finish and **fetching** on the next clock cycle, both are done simultaneously.
{% endstep %}

{% step %}
### **Instruction 2: LDI R17, 0x03**

While the previous instruction is being executed, the CPU fetches the next instruction `LDI R17, 0x03` from flash again —  this time from `0x0001` instead of `0x0000` — and saves it to the instruction register. The program counter increments again from `0x0001` to `0x0002`.

#### <mark style="color:$success;">**Decode**</mark>

**Decode:** The control unit reads from the instruction register and sees `1110 0000 0001 0011`. Again, the control unit identifies the binary as an LDI instruction with a constant `0x03` being written into `R17`.

#### <mark style="color:$danger;">**Execute**</mark>**&#x20;+** <mark style="color:blue;">**Fetch**</mark>

The control unit runs the operation, and `0x03` actually gets saved to `R17`.
{% endstep %}

{% step %}
### **Instruction 3:** ADD R17, R16

Now, the fetch for the next instruction. The CPU reads the instruction in flash at the address `0x0002` and saves it to the instruction register. The program counter then increments again to `0x0003`.

#### <mark style="color:$success;">**Decode**</mark>

**Decode:** The control unit reads from the instruction register and sees `0000 1111 0001 0000`. Looking at the [instruction set manual](https://ww1.microchip.com/downloads/en/DeviceDoc/AVR-InstructionSet-Manual-DS40002198.pdf) again, the mask for an ADD operation is `0000 11rd dddd rrrr`.

* `0000 11` lets the control unit identify that the operation is an `ADD` operation.
* `r ... rrrr` is the source register
* `d dddd` is the destination register

Notice that this time instead of four bits for a register, we actually get five. This means that we can use the full set of registers — `R0` to `R31`. Now, binary `00000` means `R0`, `00001` means `R1`, `00010` means `R2`, and so forth. Looking at our binary and the format again, let's break it down.

`0000 1111 0001 0000`

`0000 11rd dddd rrrr`&#x20;

Combining `r ... rrrr` we get `10000`, which is register `R16`. Thus, the source is `R16`.

Combining `d dddd` we get `10001`, which is register `R17`. Thus, the destination is `R17`.

Now the control unit knows:

* The instruction is `ADD`
* The source is `R16`
* The destination is `R17`

#### <mark style="color:$danger;">**Execute**</mark>

The control unit runs the operation; the Arithmetic Logic Unit (ALU) performs 5+3, and the output is saved to register `R17`. The program is now finished.

{% hint style="info" %}
Note: Even at the end of the program, the control unit will attempt to fetch the next instruction. Thus, it is important to add an `RJMP .` at the end of a program.
{% endhint %}
{% endstep %}
{% endstepper %}
