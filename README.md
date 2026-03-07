<img src="logo.png" alt="Logo" height="60" align="right">

# *Monela OS*

<br>

[![License](https://img.shields.io/badge/license-GPL--2-blue)](https://opensource.org/licenses/GPL-2.0) 
[![Status](https://img.shields.io/badge/status-Experimental-yellow)](https://github.com/) 
[![C](https://img.shields.io/badge/-%20-3949AB?logo=c&logoColor=white)](https://en.wikipedia.org/wiki/C_(programming_language)) 
[![Rust](https://img.shields.io/badge/-%20-CE412B?logo=rust&logoColor=white)](https://www.rust-lang.org/) 
[![Linux](https://img.shields.io/badge/kernel-Linux-lightgrey?logo=linux)](https://www.kernel.org/)

## About
Monela is a Linux kernel fork that preserves full compatibility with standard Linux binaries and drivers while internally restructuring how the kernel executes tasks. Externally, it behaves exactly like Linux, so applications, libraries, and drivers work as expected. To Linux and software running on it, memory and resources appear fully global, but internally they are **owned, isolated, and verified**, providing zero-trust behavior without breaking compatibility.  

Internally, Monela divides kernel subsystems into per-core execution units. Each unit handles a portion of kernel tasks, and tasks are distributed to units in parallel using internal queues that coordinate work without shared mutable state. This reduces race conditions on multi-core processors and enforces structured ownership of resources. By keeping the split **transparent**, Linux sees normal global behavior while benefiting from per-core isolation.  

Memory management, scheduling, and device handling respect core-level ownership. Kernel functions that would normally share data across cores route requests to the owning core, preventing simultaneous conflicting access. All kernel objects are capability-tagged, meaning each subsystem can only access memory or resources it has explicit permission for. This ensures that no module can modify another module's state without proper authorization, improving fault isolation and reducing unintended interactions.  

All critical kernel structures and objects are immutable once created, preventing accidental or unauthorized modifications. Additionally, every operation is automatically reverified against ownership and capabilities, ensuring no module can bypass restrictions. Because verification is done asynchronously and per-core, **it can scale with workload without significantly impacting performance**, even under heavy system load.  

---

## Features

- **Per-core kernel subsystems:** Each major subsystem is pinned to a CPU core to minimize shared-state conflicts and maximize parallel execution. Linux sees everything as normal global memory.  
- **Dynamic task scaling:** When workloads increase, additional instances of a subsystem are spawned automatically to handle tasks in parallel. Once the load subsides, extra instances are removed, freeing resources. Each instance operates independently, and failures are contained without affecting the main execution flow.  
- **Parallel task execution:** Subsystems receive tasks via internal queues and execute them concurrently across cores, without requiring inter-core message passing or global locks.  
- **Ownership-based memory and resource management:** Kernel objects are assigned owners and capability tags to prevent unauthorized access across cores.  
- **Full Linux compatibility:** Maintains standard Linux syscall behavior, filesystem access, drivers, and libraries.  
- **Hardware-aware scaling:** The number of dynamic execution units is limited by available CPU cores, memory, and cache, ensuring safe operation under all supported hardware configurations.  
- **Immutable core structures:** Critical kernel objects cannot be altered after creation, guaranteeing system integrity.  
- **Per-operation verification:** Every action is automatically checked against ownership and capability rules, preventing unauthorized changes. Verification is done asynchronously to maintain performance.  
- **Application Base:** Monela uses APT. 
- **Sudo is hardened:** Sudo operations are more heavily managed.  

---

## Hardware Requirements

Monela is designed for modern multi-core hardware. Using older or single-core processors may reduce performance or cause instability.  

- **CPU:** 64-bit, multi-core processor, 4 cores or more recommended. Tested on Intel Core i5/i7 (2010+) and AMD Ryzen (2017+).  
- **GPU:** Supported by Linux kernel 6.x+ drivers. Tested with NVIDIA 10-series and newer, AMD Vega/RDNA and newer.  
- **RAM:** 8 GB minimum, 16 GB recommended for full performance.  

Legacy devices or drivers that rely on global kernel access may not function correctly under Monela's per-core execution model.

## IMPORTANT 
A major misconception regarding Monela is that it functions as a "bridge" or "shim" over the Linux kernel, or that achieving full compatibility is impossible. To clarify: Monela is NOT a separate layer or a departure from the Linux architecture. It is, 100% natively, the exact monolithic Linux kernel.

Now, of course it has literal code modifications to the kernel, but it behaves the same with added enforcements and "tolls." 

All behaviors, system calls, and driver interactions remain identical to upstream Linux. The differentiation lies entirely in how the kernel is compiled and the internal logic used to process tasks. Monela preserves the standard Linux environment while only adding it's special Monela implications.  

Simply, the INPUT/OUTPUT remains 100% the same. The only modification is how the kernel is compiled and how the INPUT reaches the OUTPUT state. 

As of right now the current development plan of Monela is to wait for the official release of Linux 7 that will support Rust fully. Then I will use Rust and write a second kernel that only has the needed functions and features described above and then I will compile it with the Linux kernel. This prevents errors caused from mixing code into the Linux kernel. It is planned to be a dual kernel OS. This does sound impossible, but it's not as unrealistic as it may sound. 


DOCS ARE COMING SOON (40 PAGE PDF), CHECK BACK LATER FOR MORE.
