This repository is for the semicon 2026.


Here are some usefull links (they are also downloaded and in this repository, but if you want to look at the original repository, here are the links).
-	Secure boot and update with PQC is demonstrated in this code example - https://github.com/Infineon/mtb-example-ce243041-secure-boot-and-update-with-epb/
-	Bootloader (Edge Protect Bootloader) with PQC capability can be found here: https://github.com/Infineon/mtb-example-ce43373-edgeprotect-bootloader
-	Getting-started-psoc-control-c3pm6-security AN: https://www.infineon.com/assets/row/public/documents/non-assigned/42/infineon-an243370-getting-started-psoc-control-c3pm6-security-applicationnotes-en.pdf
-	psoc-control-c3pm6-provisioning Spec for advanced information (optional material for this hackathon): https://www.infineon.com/assets/row/public/documents/non-assigned/42/infineon-an243376-psoc-control-c3pm6-provisioning-applicationnotes-en.pdf

If you want to contact me, just write me/us, just write me an e-mail or create an issue in this repository. :)

My e-mail is: ericjoerg.schulze@infineon.com


SEMICON 2026 — PQC Hackathon Challenge
Make an ECC-Based Embedded SPDM Device Quantum-Resistant
The challenge: Migrate an ECC-based SPDM device on the PSOC™ Control C3M6 toward post-quantum protection. Secure both the boot chain and the authenticated communication channel between the device and its host.

Scope: The implementation target is PQC, not a hybrid ECC-plus-PQC protocol flow. Any classical trust anchor that remains in the immutable BootROM must be clearly identified. PQC verification by the bootloader alone does not make the entire boot chain quantum-resistant.

Why it matters
SPDM (Security Protocol and Data Model) authenticates hardware components and establishes protected sessions. The supplied code example uses ECDSA for authentication and ECDHE for key establishment. SPDM 1.4 introduces post-quantum mechanisms, including ML-DSA and ML-KEM.

Teams must map the existing ECC flow to a justified CNSA 2.0 target profile. Verify the required algorithms and security category rather than assuming that example parameter sets are appropriate. CNSA 2.0 is a design constraint for this challenge, not a claim of certification.

Platform and starting point
AREA
REQUIREMENT
MCU
PSOC™ Control C3M/P6, PSC3M6GES3A HQ1 configuration
Hardware
PSOC™ Control C3M6 Evaluation Kit (KIT_PSC3M6_EVAL)
Development
ModusToolbox™ 3.8 or later, kit BSP, and supplied host tools
Tooling
Infineon tools and supplied Infineon software
Starting code
SPDM Code Example with a PSoC responder, host requester, ECDSA credentials, ECDHE session establishment, and encrypted application traffic


Key references: Infineon AN243370, AN243376, AN243402, and the PSOC™ Control C3M/P6 MCU Architecture Reference Manual, section 15; DMTF DSP2072; SPDM 1.4; CNSA 2.0; and applicable NIST FIPS 203/204/205 standards.

Implementation tasks
TASK
WEIGHT
EXPECTED RESULT
1. Establish the ECC baseline
11.25%
Document the boot chain and lifecycle state. Demonstrate ECC-based boot and SPDM operation, including rejection tests. Record initial resource and performance figures.
2. Protect boot-time operations
15%
Use or extend the Edge Protect Bootloader (EPBL) to verify the next application image with a PQC signature before execution. Reject corrupted, incorrectly signed, malformed, and unauthorized rollback images.
3. Protect SPDM operations
30%
Implement responder authentication with ML-DSA and, where supported by the selected profile, key establishment with ML-KEM. Demonstrate host validation, session-secret derivation, and safe handling of PQC objects in MCU memory.
4. Connect the device lifecycle
18.75%
Show secure firmware and credential updates, rotation or revocation, rollback protection, and how the SPDM identity relates to the verified boot identity.
5. Present and demonstrate
25%
Explain the architecture, algorithm choices, limitations, and measured costs. Demonstrate normal operation, tamper rejection, an update, and reconnection.


Boot-chain trust boundary
Document ROM_BOOT, FLASH_BOOT, EPBL, and the application, including which stage authenticates the next image. EPBL is the integration point for PQC application-image verification. Do not assume that immutable BootROM supports ML-DSA unless the selected platform configuration explicitly demonstrates it.

Explain how EPBL itself is protected, where its PQC verification key is provisioned or stored, and which classical dependencies remain. Address MCUboot metadata, image versions, lifecycle states, and rollback protection.

Validation and live demonstration
Demonstrate both positive and negative cases:

Boot: Start authorized ECC-baseline and PQC-verified images; reject modified metadata, invalid signatures, and unauthorized rollback.
SPDM: Establish baseline and migrated sessions; authenticate the responder and exchange authenticated, encrypted application data.
Tampering: Reject altered credentials, handshake messages or transcripts, and malformed or unsupported PQC key material—without silently falling back to classical-only mode.
Update: Apply a valid firmware or credential update, reboot, and successfully reconnect.
Measurements
Compare the ECC baseline and PQC implementation, separating boot-time and runtime costs.

CATEGORY
REPORT
Security
Authentication at each boot stage; image-signature, SPDM-authentication, and key-establishment algorithms
Memory
EPBL and application flash; boot and runtime RAM; credential storage; working and transcript buffers; peak handshake usage
Performance
Image-verification and boot times; session setup; signature verification; KEM encapsulation and decapsulation
Communication
Handshake message count and total size; largest message; fragmentation behavior
Migration
Remaining classical dependencies and the upgrade path for deployed ECC devices


Report the total memory impact, not just individual key sizes.

Deliverables
Source code for the EPBL integration, PSoC responder, and host requester/test application.
Reproducible ModusToolbox build and run instructions.
ECC-baseline and PQC-implementation configurations.
Credential-generation and provisioning scripts.
Positive and negative boot and SPDM test results.
Boot-time and runtime resource and performance comparisons.
A short migration architecture identifying remaining classical trust anchors and the path toward fully PQC-protected operation.
A five-minute live demo showing verified boot, protected traffic, tamper rejection, a valid update, and successful operation afterward.
Evaluation focuses on security correctness, a credible ECC-to-PQC migration, and a robust, reproducible implementation within MCU resource constraints.
