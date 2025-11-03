# Supporting x86-64 Address Translation for 100s of GPU Lanes
- **Authors**: Jason Power Mark, D. Hill, David A. Wood
- **Venue / Year**: HPCA 2014
- **Tags**: #gpu #unified virtual address # tlb #page table walker #page walk cache
- **Updated**: 2025-11-03

---

## TL;DR
This paper analyzes how real GPU workloads stress the MMU and TLB hierarchy.  
It proposes a **highly-threaded page table walker** and a **page walk cache (PWC)** to handle massive GPU parallelism and high TLB miss rates efficiently.  
The final design achieves near-ideal translation performance while maintaining compatibility with x86-64 virtual memory.

---

## Problem & Motivation
- Modern GPUs (pre-Pascal) lacked full virtual memory compatibility with CPU which limited heterogeneous execution and memory sharing. 
- **Unique Challenge:** Conventional CPU MMU architecture are unsuitable for GPUs due to **extreme thread-level parallelisim**. 
- This paper aims to design a GPU MMU to support x86 address translation efficiently.

---

## Key Ideas / Method
- **Design 0 (CPU-style MMU)** – Per-core MMU similar to CPUs.  
  Consumes too much power and is inherently serial, making it unsuitable for massively parallel GPUs.

- **Design 1** – Each Compute Unit (CU) has a private L1 TLB and a page walker.  
  Queuing delay becomes extremely long when many warps miss simultaneously.

- **Design 2 (Highly-threaded Page Table Walker)** – Introduces a multi-threaded walker that can handle many in-flight page walks in parallel.  
  However, TLB miss rate remains high and each walk still requires traversing four levels of page tables.

- **Design 3 (Page Walk Cache, PWC)** – Adds a cache for intermediate page table entries, reducing repeated memory accesses and page walk latency.  
  Achieves performance within 1–2% of an ideal MMU.

- **Other Designs** – Explores *Shared L2 TLB*, *TLB prefetching*, and *Large pages*.  
  These methods are complex or workload-sensitive, and while they perform poorly in this paper’s evaluation, many concepts were later adopted by modern GPUs (e.g., banked L2 TLBs, prefetchers, multi-page support).

---

## Implementation / Experimental Setup
- Built on a modified gem5–GPGPU-Sim full-system simulator supporting x86-64 page tables.  
- Rodinia benchmark and a sort program.
- Compared to ideal MMU.

---

## Evaluation & Results
- Design 3 (with PWC) achieves performance within 1–2% of an ideal MMU. 
- The paper focuses more on **architectural feasibility and system correctness** than raw speedup.  
- The main takeaway is that GPU MMUs can be made compatible with x86-64 virtual memory without significant performance loss.

---

## Strengths
- **First comprehensive analysis** of GPU MMU behavior under real workloads, revealing how massive thread-level parallelism stresses TLBs and page walkers.  
- **Systematic exploration of multiple MMU designs (Designs 0–3)**, showing how parallel page walkers and page walk caches improve scalability.  
- **Architecturally influential work** — laid the foundation for later unified virtual memory (UVM) systems and inspired features like shared L2 TLBs, multi-level PWCs, and hardware page-fault handling in modern GPUs.


---

## Limitations / Open Questions
- The evaluation focuses on early GPU workloads (Rodinia) and lacks modern ML or data-intensive applications.  
- Designs such as the **shared L2 TLB** and **large page support** were evaluated simplistically and may not reflect modern implementations.  
- Does not explore **OS-level page migration** or dynamic memory management across CPU–GPU contexts — topics later addressed by NVIDIA UVM and AMD HMM.  
- Open direction: extending GPU MMU research to **machine learning and large-memory workloads** with adaptive page sizing and intelligent TLB coalescing.

---

## My Takeaways
I was surprised that GPUs in 2014 still lacked virtual memory support.  
What I find most interesting is how the paper looks at MMU design from a **shared CPU–GPU perspective**, adapting CPU-style page tables to GPU-level parallelism.  
The role of the **memory coalescer** is also striking — by merging memory requests across lanes, it reduces TLB pressure and improves throughput even if individual accesses become slightly slower.  
This idea of balancing latency and throughput remains central in today’s GPU memory hierarchy.

