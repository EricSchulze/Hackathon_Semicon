# SEMICON 2026 | PQC Hackathon Challenge

## Make an ECC-Based Embedded SPDM Device Quantum-Resistant

This repository contains the challenge materials, code examples, and references for the SEMICON 2026 PQC Hackathon.

## Why This Challenge Matters

Public-key cryptography is entering a formal migration phase. Future quantum computers could break the assumptions behind widely deployed elliptic-curve cryptography. Data captured today may also be stored and decrypted later, so migration planning cannot wait until a cryptographically relevant quantum computer exists.

CNSA 2.0 is the primary driver for this challenge. For applicable U.S. national-security systems and their suppliers, it defines a transition away from quantum-vulnerable public-key algorithms such as ECDSA and ECDH. Migrating an existing product also requires preserving device identity, updateability, interoperability, performance, and operation within constrained memory.

This challenge focuses on **SPDM (Security Protocol and Data Model)**, the DMTF standard for authenticating hardware components and establishing secure sessions. In datacenters, devices such as GPUs, storage controllers, PCIe/CXL add-in cards, and SmartNICs/DPUs can act as SPDM responders. Platform and management controllers, such as BMCs, act as requesters that verify device identity and firmware measurements before trusting a component.

**SPDM 1.4 adds post-quantum mechanisms**, including ML-DSA signatures and ML-KEM key establishment, alongside classical ECDSA/ECDHE mechanisms. This challenge asks teams to migrate an ECC-based SPDM device on real MCU hardware and measure the engineering costs.

## Regulatory and Standards Context

Participants must distinguish binding requirements from guidance and standards:

- **CNSA 2.0** is the primary challenge driver and target profile. State the applicable requirements and map the existing ECC flow to the selected post-quantum algorithms.
- **DMTF DSP2072, “SPDM Post-Quantum Cryptography White Paper,”** is an informative and directly relevant migration reference.
- **U.S. Executive Order 14028 and National Security Memorandum 10** drive federal cryptographic inventory and post-quantum migration planning. They do not require every commercial product to replace ECC immediately.
- **NIST FIPS 203, 204, and 205** standardize ML-KEM, ML-DSA, and SLH-DSA, respectively. Their relevance depends on the selected implementation and applicable requirements.
- **NIST IR 8547** describes the transition from quantum-vulnerable to post-quantum standards and is a planning reference, not a standalone law.
- **The EU Cyber Resilience Act, NIS2, and DORA** establish cybersecurity, lifecycle, and risk-management obligations; they do not universally prescribe replacing ECC with PQC.
- **BSI and UK NCSC guidance** recommends cryptographic inventory, prioritization, and migration planning for systems with long confidentiality or service lifetimes.

For this hackathon, state the CNSA 2.0 profile being targeted, show how it maps to the existing ECC flow, and identify any remaining classical dependencies. CNSA 2.0 is a design constraint for this challenge, not a claim of certification.

## Starting Point

### Device and Toolchain

| Area | Requirement |
|---|---|
| MCU | PSoC™ Control C3M/P6, using the `PSC3M6GES3A HQ1` configuration |
| Evaluation kit | PSoC™ Control C3M6 Evaluation Kit (`KIT_PSC3M6_EVAL`) |
| Development environment | ModusToolbox™ 3.8 or later, the kit BSP, and supplied host-side tools |
| Tooling constraint | Use Infineon tools and supplied software, including ModusToolbox, its bundled compilers, Edge Protect tooling, the kit BSP, and the provided examples |

### Platform and Secure-Boot References

Use the supplied platform documentation as the reference for device behavior, provisioning, and secure boot:

- **AN243370, Getting Started with PSoC™ Control C3x6 MCU Security:** secure boot, boot flow, EPBL, image signing, lifecycle security, firmware upgrades, and key revocation.
- **AN243376, PSoC™ Control C3x6 MCU Provisioning Specification:** OEM root-of-trust provisioning, policy configuration, image signing, re-provisioning, lifecycle transitions, and key revocation.
- **AN243402, Getting Started with Trusted Firmware-M on PSoC™ Control C3:** TF-M and EPBL integration, image signing and programming, secure boot, and image upgrades.
- **PSoC™ Control C3M/P6 MCU Architecture Reference Manual, Section 15:** ROM_BOOT, FLASH_BOOT, image authentication, MCUboot-format images, OEM root-of-trust keys, lifecycle states, and secure-boot behavior.
- **PSoC™ Control C3M/P6 getting-started documentation:** evaluation-kit setup, BSP usage, ModusToolbox workflow, and supported platform cryptography.

The platform documentation identifies the **Edge Protect Bootloader (EPBL)** as the integration point for PQC image verification and describes ML-DSA image-validation configuration. Use or extend EPBL to verify the next application image with the selected PQC signature before execution. State which stages are authenticated by immutable BootROM and which by EPBL. Do not assume that immutable BootROM supports ML-DSA unless the selected configuration explicitly demonstrates it.

### SPDM Code Example

The supplied **SPDM Code Example (SPDM CE)** is the real-time communication baseline. It provides:

- An SPDM responder running on the PSoC device
- A host acting as the SPDM requester
- ECDSA device authentication and ECC-based certificates or trust anchors
- ECDHE session-key establishment
- Authenticated and encrypted application data after session establishment

The example is functional but uses quantum-vulnerable public-key cryptography. The migration target is the post-quantum capability defined by SPDM 1.4.

## Challenge Objective

Design, implement, and demonstrate a two-part migration:

1. **Boot-time protection:** Protect the chain of trust from reset through application startup, including verification of the next application image.
2. **Real-time protection:** Protect device-to-host communication with SPDM after the application starts.

Both parts are mandatory. A PQC-protected SPDM session is incomplete if an unauthorized application can boot. Secure boot is incomplete if the running application exposes only a classical or unauthenticated runtime channel.

The challenge target is **not a hybrid ECC-plus-PQC protocol flow**. Use the selected PQC algorithms in the migrated target. The ECC implementation is the baseline for comparison. Clearly document any classical trust anchor that remains in immutable BootROM or another earlier boot stage. Do not claim that the entire chain is quantum-resistant while such a dependency remains.

## Required Implementation Tasks

### Task 1: Establish the ECC Baseline (11.25%)

Use the supplied ECC-based boot and SPDM implementation as the reference configuration.

Document the device boot chain, including ROM_BOOT, FLASH_BOOT, EPBL or the application stage, the OEM root-of-trust key, image format, and lifecycle state. Demonstrate applicable next-stage image authentication, rejection of an altered or incorrectly signed image, and the configured development or production lifecycle behavior.

Demonstrate the runtime flow:

- PSoC device acting as the SPDM responder and host acting as the requester
- Responder authentication with ECDSA credentials and certificates
- Session establishment using ECDHE
- Authenticated and encrypted application-data exchange
- Operation in a development lifecycle state suitable for an open hackathon

Record baseline flash and RAM usage, session-establishment time, message sizes, and application throughput or latency.

### Task 2: Protect Boot-Time Operations (15%)

Use or extend EPBL so that the next application image is authenticated with a PQC signature before execution.

Document and test:

- The separation between immutable BootROM verification and EPBL verification
- The boot chain and the exact stage at which each image is authenticated
- MCUboot metadata, image versions, and rollback protection
- The PQC image-validation key and how it is provisioned or embedded
- The selected algorithm and its mapping to the stated CNSA 2.0 target
- Rejection of corrupted, incorrectly signed, malformed, and unauthorized-rollback images
- The migration path from the existing ECC root of trust, including any classical dependency that remains

Explain how EPBL itself is authenticated and protected. If its authenticity ultimately depends on a classical immutable root of trust, document that limitation rather than claiming the entire chain is PQC-protected.

### Task 3: Protect Real-Time Operations with SPDM (30%)

Extend the baseline so that the responder can be authenticated and a session established with post-quantum protection.

The implementation should use:

- **ML-DSA** for post-quantum authentication and signatures, as defined by the applicable SPDM 1.4 profile
- **ML-KEM** for post-quantum key establishment where supported by the selected SPDM or library profile
- A certificate, credential, or trust-anchor format that the host can validate

Map the selected algorithms to the stated CNSA 2.0 target. Do not assume that ML-DSA-65 or ML-KEM-1024 is automatically the correct profile; verify the required security category and algorithm selection.

Address host validation, session-secret establishment and derivation, credential and trust-anchor provisioning, and the memory impact of PQC keys, signatures, certificates, and transcripts.

### Task 4: Connect the Device Lifecycle (18.75%)

Show that boot-time and runtime protection form one maintainable device lifecycle. Implement or describe a secure update path for application images and SPDM credentials. Demonstrate acceptance of a valid update and rejection of corrupted, incorrectly signed, or unauthorized-rollback updates.

Explain the relationship between the verified boot identity and the SPDM responder identity. State whether the runtime identity is anchored in the verified boot chain, separately provisioned credentials, or both, and describe the security consequences.

Include a migration strategy for devices that initially contain ECC credentials. Identify which roots of trust remain classical, which components are PQC-protected, and what must change before the product can claim full quantum-resistant protection.

### Task 5: Present and Demonstrate the Migration (25%)

Prepare a concise technical presentation covering:

- The original ECC architecture and trust model
- The selected post-quantum algorithms and their mapping to the CNSA 2.0 target
- The roles of BootROM, FLASH_BOOT, and EPBL
- The SPDM authentication and key-establishment migration
- Provisioning, credential management, updates, revocation, and rollback protection
- Remaining classical dependencies and limitations
- Resource, performance, and communication costs versus the ECC baseline
- Engineering trade-offs and recommendations for future migration phases

The live demonstration must show an authorized image booting, rejection of an invalid or rollback image, successful SPDM responder authentication, a protected session, authenticated and encrypted application communication, a tampering attempt and its rejection, and successful operation after a valid update.

## Validation Scenarios

Run positive and negative tests for both the ECC baseline and migrated design.

**Positive tests:**

- ECC baseline image verification and boot
- PQC next-image verification and boot through EPBL
- Valid firmware or credential update followed by successful reboot
- ECC baseline SPDM session establishment
- Migrated SPDM session establishment and responder authentication
- Successful application-data exchange and reconnect after update

**Negative tests:**

- Modified responder certificate or credential
- Invalid signature
- Modified MCUboot image metadata or manifest
- Unauthorized rollback image
- Modified SPDM transcript or handshake message
- Unsupported or malformed PQC key material
- Downgrade attempt or fallback to classical-only operation

## Measurements and Comparison

Compare the ECC baseline and PQC implementation, separating boot-time and runtime costs.

| Category | Required measurements |
|---|---|
| Security | Authentication at each boot stage; image-signature, SPDM-authentication, and key-establishment algorithms |
| Memory | BootROM, EPBL, and application flash; boot-time and runtime RAM; credential storage; working buffers; peak handshake memory |
| Performance | Boot verification and boot-to-application times; session setup; signature verification; KEM encapsulation and decapsulation |
| Communication | Handshake message count and total size; largest message; fragmentation behavior |
| Migration | Remaining classical dependencies in the boot chain and SPDM; components changed to PQC; upgrade path for deployed ECC devices |

Report total memory impact, not just individual key sizes. Include certificates, signatures, transcript and temporary computation buffers, metadata, and persistent trust anchors.

## Expected Deliverables

1. Source code for the EPBL integration, PSoC responder, and host requester/test application
2. Reproducible ModusToolbox build and run instructions
3. ECC-baseline and PQC-implementation configurations
4. Credential-generation and provisioning scripts
5. A test report covering positive and negative boot-time and SPDM scenarios
6. A boot-time and runtime resource and performance comparison
7. A migration architecture describing remaining classical trust anchors and the path toward fully PQC-protected operation
8. A five-minute live demo showing verified boot, protected traffic, tamper rejection, a valid update, and successful operation afterward

## Evaluation Criteria

### Security Correctness

- Only authorized images boot; corruption, invalid signatures, and unauthorized rollback are rejected.
- The requester authenticates the responder, and the session provides authenticated encryption for application data.
- Invalid credentials, signatures, transcripts, and downgrade attempts are rejected.
- The implementation does not silently fall back to classical-only operation.

### Migration Quality

- The design addresses a real ECC-to-PQC transition.
- The stated compliance driver is mapped to concrete technical changes.
- Remaining classical dependencies and transition limitations are clearly identified.
- Provisioning, rotation, revocation, firmware updates, lifecycle transitions, and rollback protection are considered.

### Embedded Engineering

- The implementation fits PSoC resource constraints or clearly documents trade-offs.
- Large PQC objects are handled safely, with bounded memory use.
- The implementation is reproducible, understandable, and testable.

## Useful Links

Some referenced materials are also available in this repository. These links point to the original sources:

- [Secure boot and update with PQC (CE243041)](https://github.com/Infineon/mtb-example-ce243041-secure-boot-and-update-with-epb/)
- [Edge Protect Bootloader with PQC capability (CE43373)](https://github.com/Infineon/mtb-example-ce43373-edgeprotect-bootloader)
- [AN243370: Getting Started with PSoC Control C3M6 Security](https://www.infineon.com/assets/row/public/documents/non-assigned/42/infineon-an243370-getting-started-psoc-control-c3pm6-security-applicationnotes-en.pdf)
- [AN243376: PSoC Control C3M6 Provisioning](https://www.infineon.com/assets/row/public/documents/non-assigned/42/infineon-an243376-psoc-control-c3pm6-provisioning-applicationnotes-en.pdf)

## Contact

Questions? Email me or open an issue in this repository.

[ericjoerg.schulze@infineon.com](mailto:ericjoerg.schulze@infineon.com)
