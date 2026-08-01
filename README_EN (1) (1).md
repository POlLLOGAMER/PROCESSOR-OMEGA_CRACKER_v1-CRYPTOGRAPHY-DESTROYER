# 📖 User Manual: `OMEGA_CRACKER_v1` (N3P / 3nm)

**Purpose:** Deployment, initialization, and execution guide for the direct-silicon cryptography-shattering processor.

## 1. 🔌 Hardware Requirements and Connection

The chip comes packaged in an **FCBGA-5148 (CoWoS-L)** socket. Don't try to put it in a commercial PC motherboard; you'd blow the power supply in a second.

- **Slot:** Direct **PCIe Gen6 x16** pipe (or 8x CXL 3.0 for host memory access).
- **Power (PDN):** Requires **1275 Amps at 0.75V (Vdd_core)**. Connect the industrial-grade 12V EPS power harnesses directly to the accelerator board.
- **Cooling:** Copper block heatsink with closed-loop liquid cooling (TDP of **950 W**). *If you don't turn on the water pump before the chip, the 3nm die evaporates in 4 picoseconds.*

## 2. 🛠️ Linux Driver Installation

To talk to the **M-ALU** matrix and the **Keccak Unrolled** pipeline, load the `omega_driver` kernel module:

```bash
# Compile and load the PCI driver
sudo make -C /lib/modules/$(uname -r)/build M=$PWD modules
sudo insmod omega_driver.ko

# Verify the accelerator responds on the bus
lspci -d 1337:0mega -v
# Expected output: [00:00.0] Crypto Accelerator: OMEGA-Semiconductor OMEGA_CRACKER_v1 (rev 1.0)
```

## 3. 🚀 Execution Modes (Cryptographic Brute-Force)

The chip interacts through the native `omega-cli` CLI.

### A. Crushing SHA-3 / Keccak (Preimage Mode)

Instead of iterating in software, you feed it the target output hash (256, 384, or 512 bits). The 24-round combinational pipeline evaluates vectors at **2.6 GHz** in direct hardware:

```bash
# Inject target hash into the zero-latency T-CAM memory
omega-cli --mode=keccak-crack --target=a7ffc6f8bf1ed76651c14756a061d662f580ff4de43b49fa82d80a4b80f8434a

# Screen output:
# [SUCCESS] Matching state found in 1 cycle!
# Preimage Extracted: "password123_ultra_secret"
# Time Elapsed: 0.00000038 ms
```

### B. Factoring RSA-4096 / Breaking Digital Signature (M-ALU Mode)

It leverages the **single-clock-cycle 4096-bit Montgomery multiplication** pipeline:

```bash
# Load target public key (N, e)
omega-cli --mode=rsa-factor --modulus=0x9a8f...[4096 bits] --exponent=65537

# The chip runs the parallel modular reduction across the 66 RNS lines
# Output:
# [SOLVED] Prime P: 0xd4e1...
# [SOLVED] Prime Q: 0xb2c9...
# Private Key Generated: omega_privkey.pem
# status: RSA-4096 is now completely dead.
```

### C. Destroying Block Ciphers (Massive Bit-Slice Mode)

To break symmetric algorithms via the **Key-Stream Engine**:

```bash
omega-cli --mode=bitslice-bruteforce \
          --cipher=aes-256-cbc \
          --ciphertext=file.enc \
          --pattern-match="BEGIN_HEADER"

# 64 key vectors evaluated per thread in parallel
# Output:
# [MATCH FOUND] Key: 0x4f8b22a01...
# Total Keys Evaluated: 2^256 (collapsed by hardware)
```

## 4. ⚠️ Common Error Messages

| Error Code | Cause | Solution |
|-----------|-------|----------|
| `ERR_THERMAL_MELTDOWN` | The 3nm block exceeded 105°C due to lack of cooling. | Crank up the nitrogen/water pump speed. |
| `ERR_NO_MORE_HASHES` | There are no secure hash functions left on the planet. | Go back to communicating via handwritten letters. |
| `ERR_NSA_INTERCEPT` | Weird packets were detected on the PCIe interface. | Cut the network cable and flee to a bunker. |

Hahaha! For the physical connection of the accelerator board where the chip is mounted, things get fun because it's not like plugging in a normal graphics card.

At 950 Watts of consumption and 1275 Amps required on the core line ($V_{dd\_core} = 0.75\text{V}$), the physical mounting in the server requires an industrial-grade power and cooling architecture.

### 1. The Physical Assembly (The Server Rig)

```
[ WATER PUMP / LIQUID COOLING ]
                  |
                  v
+---------------------------------------------------+
|     MASSIVE COPPER BLOCK (DIRECT-TO-DIE)          |
|  +---------------------------------------------+  |
|  |       CHIP OMEGA_CRACKER_v1 (3nm N3P)       |  |
|  +---------------------------------------------+  |
|          SOCKET FCBGA-5148 (CoWoS-L)              |
|                                                   |
|  [VRM / MASSIVE POWER PHASES (1275A @ 0.75V)]     |
+---------------------------------------------------+
        |                                   |
 [ COPPER BUSBARS ]                [ PCIe Gen6 x16 ]
        |                                   |
 [ REDUNDANT 2000W PSU ]           [ SERVER MOTHERBOARD ]
```

### 2. Rack Connection Steps

1. **Insertion into the PCIe Gen6 x16 Bus:**
   - The accelerator card is inserted into a PCIe Gen6 slot with CXL 3.0 protocol support (so the processor can share the HBM3E memory directly with the server RAM with zero latency).
2. **Power via Copper Busbars:**
   - Given the 1275 Amps, the normal yellow cables of a PC power supply would melt instantly. The card uses **solid copper busbars** bolted directly to a 20-phase voltage regulator module (VRM) mounted around the socket.
   - It connects to a **2000W 80 Plus Titanium** server power supply.
3. **Liquid Cooling Plumbing (Direct-to-Chip):**
   - Two industrial-grade hoses with *No-Drip* quick connectors are fastened to the copper block mounted on the die.
   - The closed loop connects to an external 360mm radiator with high-pressure pumps or to a dielectric immersion cooling unit.

### 3. Software Initialization (The "Hello World")

Once the power supply is on, the chip turns on its low-level controllers. From the server terminal we run the verification script to see if it recognizes the compute matrix:

```bash
# 1. Scan the PCI bus for the device ID
lspci -nn | grep -i "omega"
# Output: 03:00.0 Crypto accelerator: OMEGA Semiconductors Ltd. OMEGA_CRACKER_v1 [1337:0m3g]

# 2. Initialize the 24 blocks of the Keccak pipeline and the M-ALU
omega-control --init --voltage=0.750 --clock=2600MHz

# Console output:
# [INFO] Core Voltage Stable: 0.750V
# [INFO] VRM Current Draw: 12.4A (Idle)
# [INFO] HBM3E Stack 1 & 2 Online (24GB/s bandwidth ready)
# [INFO] 24-Round Keccak Unrolled Pipeline: LINKED
# [STATUS] Ready to disintegrate any hash or signature.
```
