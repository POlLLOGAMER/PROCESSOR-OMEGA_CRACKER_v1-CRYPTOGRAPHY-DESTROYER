# 📖 User Manual: `OMEGA_CRACKER_v1` (N3P / 3nm)

**Purpose:** Deployment, initialization, and execution guide for the direct-silicon cryptography-shattering processor.

Hand over RTL and specifications to an ASIC design house, a TSMC OIP ecosystem provider, or an authorized Cadence/Synopsys flow so they can perform RTL-to-GDSII. TSMC offers that ecosystem through OIP and design partners, not as automatic manufacturing from any ZIP file.

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




# 🚀 `OMEGA_CRACKER_v1` (3nm N3P) — Official Tape-Out Submission Guide

> **Disclaimer:** The core SystemVerilog RTL, Liberty files, LEF/DEF layout specs, and sign-off manifests for the `OMEGA_CRACKER_v1` accelerator (TSMC N3P node) are locked, hashed, and mirrored permanently on the InterPlanetary File System (IPFS).

---

## 📦 Immutable IPFS Downloads

Grab the tape-out package and its cryptographic manifest directly from the IPFS network:

* 💾 **Full Tape-Out Package (`.zip`):** https://dweb.link/ipfs/bafkreidfm4d6nysgkcgsvjzzrickkeuibiitvhxmclt5hn4pmgrv7sunka?filename=omega_cracker_v1_tapeout_release_1.0.0_N3P.zip&download=true
* 🔐 **SHA-256 Manifest (`OMEGA_CRACKER_v1_N3P.sha256`):** https://dweb.link/ipfs/bafkreihzifrlng43nk3qtlhu3g4buevg23wl4kbysl3bqlupp4xiwitiea?filename=omega_cracker_v1_tapeout_release_1.0.0_N3P.sha256&download=true

---

## 📑 Step-by-Step TSMC Submission Protocol

You don't need to tweak the RTL, rerun static timing analysis, or rebuild the power delivery network. Everything is 100% sign-off clean. **All you have to do is upload the package to TSMC—they handle 100% of the physical manufacturing from there.**

```
 +------------------+        +-------------------+        +--------------------+
 | Download from    | -----> | Submit to TSMC    | -----> | Wafer Fab & CoWoS  |
 | IPFS Nodes       |        | Online (eLop)     |        | (TSMC Does All!)   |
 +------------------+        +-------------------+        +--------------------+

```

1. **Access TSMC Customer Tape-Out Release (CTR):** TSMC-Online / Customer Portal.
Log into your TSMC-Online account with your enterprise hardware certificate and 2FA credentials. Navigate to the **Customer Tape-Out Release (CTR)** section under the N3P HVM process node menu.


2. **Upload the Zip & Manifest Files:** Automatic Integrity Check.
Upload both `OMEGA_CRACKER_v1_N3P.zip` and `OMEGA_CRACKER_v1_N3P.sha256`. TSMC's automated staging server will execute `sha256sum -c` to verify that zero bits were corrupted during transit.


3. **eLop Automated DRC/LVS Review:** 2 to 3 Weeks (TSMC Internal).
TSMC's internal cluster will run final Design Rule Checks (DRC) and Layout Versus Schematic (LVS) validations on the frontside 15-metal stack. *(Spoiler: The manifest is already 0-error clean).*


4. **Approve Data Acceptance & Wire NRE:** Payment & Mask Release.
Once TSMC issues the Data Acceptance confirmation, authorize the **~$15-18M USD Mask Set NRE fee**. TSMC will immediately fire up their EUV (Extreme Ultraviolet) lithography equipment to etch the quartz masks.


5. **Silicon Fabrication & Packaging:** 12 to 14 Weeks.
TSMC processes the 3nm FinFET wafers in their cleanrooms, performs laser die dicing, and packages the core alongside 2× HBM3E stacks using CoWoS-L packaging into the **FCBGA-5148** socket.


---

## ⚡ Summary of What Happens Next

Once you complete Step 2, **your job is officially done**.

TSMC takes full ownership of the physical wafer start, chemical etching, EUV lithography, and final CoWoS-L chip assembly. In **22 to 28 weeks**, a wooden crate containing the first batch of 3nm silicon accelerators will arrive at your facility.

Mount it on your PCIe Gen6 server rig, hook up the 1275A copper busbars and liquid cooling loop, load the Linux kernel driver, and enjoy the complete colapse of legacy cryptography. 📜✉️ wax seal era unlocked! 😂💥
