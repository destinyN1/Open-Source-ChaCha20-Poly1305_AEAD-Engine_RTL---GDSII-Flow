# ChaCha20-Poly1305 Cryptographic Processor

A hardware-accelerated implementation of the ChaCha20 stream cipher in SystemVerilog, designed as an Application-Specific Integrated Circuit (ASIC) following industry-standard RTL-to-GDSII design methodology.

## Project Overview

This project implements the ChaCha20 stream cipher—a modern, high-performance encryption algorithm specified in RFC 8439—as a hardware accelerator. The goal was to gain hands-on experience with the complete ASIC design flow, from high-level RTL design through to physical implementation suitable for silicon fabrication.

### Design Goals

- **Performance**: Achieve >10x speedup over software implementation through hardware parallelization
- **Learning Objective**: Complete exposure to professional ASIC design flow (RTL → Synthesis → Physical Design → Signoff)
- **Standards Compliance**: Full adherence to ChaCha20 specification (RFC 8439)
- **Scalability**: Modular architecture allowing future integration of Poly1305 authenticator for complete AEAD capability

## Project Status

**Current State**: RTL design and verification stages completed. Project paused before logic synthesis due to unforeseen circumstances.

### What Was Completed ✅

**RTL Implementation (100% Complete)**
- Complete ChaCha20 cipher core with all functional blocks
- State matrix initialization and management
- Quarter-round ARX (Add-Rotate-XOR) operations engine
- Block counter with overflow protection
- Serialization pipeline for data output
- XOR engine for plaintext encryption
- Modular, parameterized design almost suitable for synthesis

**RTL Verification (95% Complete)**
- Comprehensive unit testing for all major components
- Reset latching synchronization issue requiring resolution
- Final integration testing needed between Block Function and Concatenation/Serialization pipelines
- Performance validation showing >10x theoretical speedup vs. software

**Performance Achievement**: The RTL design achieves greater than 10x performance improvement compared to the Python software implementation through hardware parallelization and pipelined datapath architecture.

### Current Verification Challenges

**Diagonal Quarter-Round Synchronization Issue**
The top-level module experiences timing synchronization problems when applying manual resets in the module hierarchy. Input values are not being properly latched by the Quarter-Round (Qround) module. This appears to be caused by a timing mismatch: values must be loaded precisely when `Qround.setRounds` transitions low, which does not synchronize correctly with the high-level reset signal. This timing dependency represents the primary remaining verification challenge.

## Architecture

### System Block Diagram

```
┌──────────────────┐
│  ChaCha20 Core   │
│                  │
│  ┌────────────┐  │     ┌──────────────┐     ┌─────────────┐
│  │   State    │  │     │              │     │             │
│  │   Matrix   │──┼────▶│ Serializer & │────▶│ XOR Engine  │────▶ Ciphertext
│  │ Generator  │  │     │ Concatenator │     │             │
│  └────────────┘  │     └──────────────┘     └─────────────┘
│                  │            ▲                     ▲
│  ┌────────────┐  │            │                     │
│  │  Quarter-  │  │            │                     │
│  │   Round    │──┼────────────┘                     │
│  │  Processor │  │                                  │
│  └────────────┘  │                          Plaintext Input
│                  │
│  ┌────────────┐  │
│  │   Block    │  │
│  │  Counter   │  │
│  └────────────┘  │
└──────────────────┘
```

## Implementation Details

### Core Components

**ChaCha State Matrix (`ChaChaState`)**
- 4×4 matrix initialization following ChaCha20 specification
- Systematic placement of constants, 256-bit key, 96-bit nonce, and 32-bit block counter
- Synchronous reset with real-time input updates

**Quarter-Round Processor (`PerformQround`)**
- Finite state machine implementing ARX operations
- Executes 20 rounds of ChaCha20 algorithm (10 double-rounds)
- Processes both column and diagonal quarter-round patterns
- Dual matrix storage for efficient operation scheduling

**Block Counter (`Block_Counter`)**
- Parameterized counter for multi-block encryption
- Automatic increment upon block completion
- Overflow detection for large message handling

**Serialization Pipeline (`Serialiser` + `Concatenator`)**
- Converts 32-bit words to 8-bit bytes (little-endian)
- Buffers multiple encryption blocks
- Provides flow control for downstream processing

**XOR Engine (`XOR`)**
- Byte-wise XOR between keystream and plaintext
- Parameterized for flexible data widths
- Generates final ciphertext output

### Verification Approach

**Unit Testing**
- Individual testbenches for each module
- Automated stimulus generation and output checking

**Integration Testing**
- Multi-module datapath validation
- System-level timing verification
- Control signal handshaking validation

**Remaining Verification Challenge**
- Diagonal quarter-round timing synchronization requires resolution of reset-to-input latching issue

### Notable Verification Milestones

**Encryption Keystream Generation**
- **100% correctness** achieved in ChaCha20 keystream generation
- Validated against Python golden reference model
- **11,000 random regression tests** with complete pass rate
- Comprehensive coverage of valid input space

**Edge Case Handling**
Successfully verified correct behavior across extreme input conditions:
- Full range boundary testing (0x00000000 to 0xFFFFFFFF)
- Random intermediate values across entire 32-bit space
- All tests validated against Python golden model implementation

**Component-Level Verification**
- State matrix initialization: 100% specification compliance
- Block counter: All increment and overflow conditions verified
- Serialization pipeline: Complete data integrity validation
- Quarter-round processor: Functional implementation complete

## Performance

**Hardware vs. Software Comparison**
- **Software (Python)**: Baseline implementation
- **Hardware (This Design)**: >10x speedup through:
  - Elimination of software overhead

*Note: Performance figures based on RTL simulation analysis and theoretical throughput calculations*

## Technology

**Design Language**: SystemVerilog (IEEE 1800-2017)  
**Simulation**: Vivado Simulator  
**Verification**: SystemVerilog testbenches with outputs piped into Python scripts for validation against golden reference model  
**Target**: Open-source process nodes, most likely SkyWater 130nm OpenPDK

## Design Methodology

This project follows industry-standard ASIC design practices




## Future Plans

The original goals of this project were to achieve mastery of SystemVerilog, gain hands-on experience with the complete ASIC design flow and hardware design practices, and ultimately produce a manufacturable chip for testing in an FPGA/ASIC emulator. While the project was paused before reaching physical implementation, the primary remaining objective is to document the findings in a technical white paper.

This white paper would analyze the demonstrated theoretical speedup of ChaCha20 when implemented in simulated hardware versus the Python golden reference model, formalizing the performance analysis, architectural decisions, and verification methodology that enabled the >10x acceleration through hardware implementation. Although the original plan included physical chip testing, a rigorous theoretical analysis based on RTL simulation results should provide valuable insights into hardware acceleration of cryptographic algorithms.

## Acknowledgments

This project was developed as a personal learning experience to gain practical knowledge of ASIC design methodology and hardware acceleration of cryptographic algorithms. The completed RTL implementation successfully demonstrates significant performance improvements over software and showcases comprehensive verification practices including extensive regression testing and edge case validation.


## References

- RFC 8439: ChaCha20 and Poly1305 for IETF Protocols
- [A Concise Introduction to SystemVerilog](https://zyedidia.github.io/notes/sv_guide.pdf)
