<img src="logo.png" alt="Logo" height="60" align="right">

# *Monela OS*

<br>

[![Status](https://img.shields.io/badge/status-Experimental-yellow)](https://github.com/) 
[![Rust](https://img.shields.io/badge/-%20-CE412B?logo=rust&logoColor=white)](https://www.rust-lang.org/) 
[![Linux](https://img.shields.io/badge/kernel-Linux-lightgrey?logo=linux)](https://www.kernel.org/)

---

## About

Monela OS is a research operating system that adds the Monela Visor, written entirely in Rust, to Linux. Monela lives beneath the Linux kernel, intercepting and mitigating all interactions between hardware, the Linux kernel, and userspace. This allows Monela to add certain features like: **monolithic compartmentalisation, workload scaling, virtual global access, CPOV (Capability, Precedence, Ownership, Verification), hsudo, verified boot, immutable system structures, and fault tolerance**.  

Linux sees standard memory, I/O, and syscalls as usual, but all processes are mediated, verified, and controlled by Monela. Global access is **moderated and restricted**, but the kernel remains monolithic from Linux’s perspective. Monela virtualises global access and enforces security without altering Linux behavior or breaking application compatibility.

This allows Monela to combine microkernel-like level isolation features, fault tolerance, and capability-based security** while maintaining full Linux compatibility, high performance, and system stability.

---

## Architecture Overview

### Monela Visor 

- **Purpose:** Monela is a hypervisor-style layer that sits below Linux. It does **not implement drivers, filesystem, or kernel services**. Its sole responsibility is to mediate, monitor, and enforce system policies.  
- **Implementation:** Written entirely in Rust for Rust's security features, speed, and reliability. 
- **Boot:**  
  1. Hardware powers on → **verified boot** → Monela Visor starts.  
  2. Monela locks immutable memory regions, sets up CPOV gates, and prepares Sub-Unit structures.
  3. Linux kernel loads above Monela; all syscalls and hardware interactions are mediated transparently.
  4. Monela sets up pre-auth channels for speed while being secure upon start up.
  5. The OS is ready to go. 
- **Immutable & Verified:** Monela control structures are immutable post-boot and the Linux kernel becomes immutable to prevent certain classes of attacks. You may enable disk encryption, key based logins, other custom security features, or SELinux policies on top of Monela to add extreme security measures. 

### Kernel Groups (KGs) and Sub-Unit Load Spawning

- **Kernel Groups:** Linux kernel subsystems are logically divided into **Kernel Groups (KGs)**, initially pinned to CPU cores for cache locality.  
- **Sub-Unit Load Spawning:** Each KG can dynamically spawn ephemeral Sub-Units to handle parallel tasks:
  - Prevents bottlenecks by avoiding a single task queue.
  - Reduces crashes: failing pipelines are automatically replaced by new Sub-Units.
  - Optimizes throughput under heavy workloads.  
- **Behavior:** While KGs are pinned to cores initially, Sub-Units **can migrate dynamically** for load balancing without breaking cache optimizations.
- **Impact:** Although Monela might be somewhat slower then standard operating systems such as Ubuntu on smaller tasks, Monela has better capability at scaling speed under heavier workloads. This is due to CPOV enforcement and puppet routing. 

### CPOV (Capability, Precedence, Ownership, Verification)

- **Purpose:** Ensures each task is verified, authorized, and routed securely.  
- **Task Types:**
  - **Macrotasks:** High-privilege operations requiring full CPOV verification (file I/O, network access, kernel-critical calls). Some macrotasks are pre-authorised and do not need to be verfiied each time. They are verified upon boot, and remain verified until a reboot where it generates new verification. This can be disabled at the cost of higher security but slower speeds. 
  - **Microtasks:** Low-privilege, frequent operations ignored by CPOV for speed (mouse interrupts, timers).  
- **CPOV Gates:**  
  1. **User-Kernel Gate:** Intercepts user requests, assigns CPOV tokens, mediates access before passing to Linux.  
  2. **Inter-KG Gate:** Allows pre-authorized communication between KGs without blocking main gate. Inter-KG macrotasks are no reverified each time, but rely on pre-auth channels verified at boot.
- **CPOV Tokens:** Every macrotask is tagged with a cryptographically verified token:
- Fields: Capability | Precedence | Ownership | Destination | Verification
Sizes: 16 bits | 8 bits | 16 bits | 16 bits | 88 bits

- Capability: Allowed operations (read/write/network/IPC/etc.)  
- Precedence: Task priority (0=background, 255=kernel-critical)  
- Ownership: Originating subsystem & instance  
- Destination: Target KG/queue  
- Verification: Signed entropy, time-bound, prevents replay attacks  
- **Sub-Unit Ports:** Each KG contains a **small verification port** that:
- Reads CPOV token, verifies authenticity, temporarily strips metadata for processing, and re-applies token for final verification.  
- **Security Outcome:** Prevents unauthorized access, enforces microkernel-style compartmentalization, mitigates race conditions, and reduces risk of kernel panics.

### hsudo (Hardened Sudo Replacement)

- Replaces standard `sudo` with **hsudo**, which:
- May require multi-factor authentication.  
- Passes requests through CPOV verification.  
- Mediates privileged operations and logs all escalations for auditability.
- As well as more uninteresting details. 

### Immutable System & Verified Boot

- Critical Monela structures are immutable.  
- Verified boot ensures Monela integrity before Linux starts.  

---

## Features

- **Below Linux:** Can intercept and mediate all kernel, hardware, and userspace interactions.  
- **Per-core Kernel Groups:** Optimizes CPU core allocation and cache usage for Linux subsystems.  
- **Sub-Unit Load Spawning:** Ephemeral parallel workers increase throughput and prevent bottlenecks.  
- **CPOV Security:** Capability-based, ownership-aware, verified task execution.  
- **Microtask/Macrotask differentiation:** Reduces overhead while preserving security.  
- **Immutable System and Verified Boot** Prevents tampering.  
- **Optional disk encryption:** Hardware-level storage security.  
- **hsudo:** Hardened, multi-factor privilege escalation.  
- **Full Linux compatibility:** Maintains standard syscall, driver, and filesystem behavior.

---

## Hardware Requirements

- **CPU:** 64-bit, multi-core (4+ cores recommended), optimized for Intel Core i5/i7 (2010+) and AMD Ryzen (2017+).  
- **GPU:** Supported by Linux kernel 6.x+ drivers (NVIDIA 10-series+, AMD Vega/RDNA+).  
- **RAM:** 8 GB minimum, 16 GB recommended.  
- Legacy devices or single-core systems may experience reduced performance or incompatibility due to per-core KG assignment.

---

## Feasibility Notes

- Monela avoids **traditional dual-kernel pitfalls** by living below Linux, mediating interactions rather than replacing it.  
- **Rust** ensures memory safety and concurrency correctness, enabling safe interception of Linux syscalls and device operations.  
- **Performance:** Light tasks may appear slightly slower than Ubuntu due to security checks, but **heavy parallel workloads scale efficiently**.  
- **Security:** Monela prevents panics, race conditions, and unauthorized access through:
- Immutable memory
- Sub-Unit redundancy
- CPOV token verification
- **Compatibility:** Linux syscalls, drivers, and applications remain fully compatible.
---

Monela OS for now is just a research operating system started by my own interest. Comibing **monolithic Linux compatibility, microkernel-style isolation, capability-based security, fault-tolerant parallel task execution, immutable & verified boot, and hardened privilege escalation**. It is **designed for high-load multi-core systems**, making it highly stable, secure, and scalable for research, enterprise, and experimental applications.

Please note: This README is an EXTREMELY light description of both WHAT Monela is and HOW Monela actually will work. More detaild documentation is currently private, but will be released in the future. (I am very busy most of the time so it's hard to find time.) 

I am also aware that having a visor be mixed in with userspace I/O among other things is extremely difficult and I am still designing how exactly this will be solved. It may in the end, change what Monela is itself. 
