---
layout: post
title: VM Isolation in Xen and KVM
categories: [All, research, vm]
tags: [vm, xen, kvm, performance isolation]
fullview: true
comments: true
---

Note: This post is based on my research when I was a grad student. The post discusses some low level details about VM, Xen, KVM, CPU scheduling. For anyone who works in this field, it could be useful.

## Introduction
The capability of running multiple jobs on a shared host has long been a design goal of modern computer systems. Enabled by today’s powerful hardware, the design of the virtual machine(VM) abstraction has been proven successful in partitioning a physical machine to support multiple user appli- cations while providing good performance, secure isolation, programmability, and flexible resource management functionalities. Popular virtual machine monitors(VMM) such as Xen [1], KVM [2], VMware ESX [3] provide the backbone for cloud computing and profoundly change the way of how people design and use IT infrastructure[4].

This article discusses the existing literature on the performance isolation topics in virtualization server environments. More precisely, the article focuses on how a guest VM’s CPU time, memory usage, IO bandwidth, and application performance degrade in an unexpected way due to the impact of other VMs. 

First, we discuss the architecture and resource models of virtualization systems and how certain design decisions lead to performance interference. We focus on two open-sourced VMMs: Xen and KVM because they are commonly used and well-studied virtualiztaion platforms. Second, we discuss in detail the known performance isolation issues of Xen and KVM and their consequences. Finally, we discuss the possible mitigations and research opportunities.

## Resource Model

According to our definition in previous section, performance interference is an un- expected event. To properly identify what is ”(un)expected”, we examine the resource models in Xen and KVM systems. We focus on three types of resources: CPU, memory, and IO. For each type of resource, we generally use the following question scheme to guide our discussion:

- How does the system divide the resource? Is it statically partitioned according to users contract, or dynamically determined according to resource availability?
- Can a VM use idle resources that exceed its entitled level?
- Does the system account the resource usage? And how?
- Is there a mechanism to enforce resource usage limit(e.g., by throt- tling)?

### CPU
In order to minimize the modifications into the guest OS, both Xen and KVM
use a two-layer scheduling design. Client applications go through two layers
of scheduling: guest-level and VMM-level scheduling. In Xen, the scheduling
entity is a guest VM while in KVM it is actually a process group . Below we discuss the VMM scheduler mechanisms and policy.

#### Xen credit based CPU scheduler
Xen uses a proportional [share based credit scheduler](http://wiki.xen.org/wiki/Credit Scheduler) for CPU allocation among VMs. The following bullet points summarize the Xen CPU scheduler:

- Administrators can specify a VM’s weight(relative) and cap(absolute). These two parameter apply to all vCPUs assigned to that VM. The
smallest allocation unit is a timeslice or quantum (30ms by default. Note: Prior to Xen v4.2 this value is not change-able. In v4.2 and above, administrators can tune this value to adapt the system for latency-sensitive workloads.). At the end of each quantum, the scheduler picks a VM’s vCPU from the ready queue and allocates the next quantum. The scheduler accounts the cumulative quantum units (or credits) and allocates quantum in a way that each VM’s count is proportional to that VM’s weight, up to its cap. Since the scheduler is non-preemptive, the assigned will not be preempted unless the vCPU blocks, yields or runs out of its quantum.

- As a VM runs tasks on its assigned vCPU, Xen accounts how much each vCPU has consumed its credit. The vCPU priority is either OVER or UNDER depending on whether it has exceeded its weighted share. A vCPU with priority UNDER is considered of higher priority and the scheduler will try to schedule it to run on any idle CPUs.

- In work-conserving(WC) mode, CPUs are only idle when there are no active VMs, meaning that a VM can consume the entire CPU while the others are idle. In non-work-conserving(WC) mode, however, VMs cannot consume more than its weighted share. So idle CPUs may not be used by active VMs.

- To guarantee fairness, the credit scheduler does its best effort to limit the CPU usage within its weighted share and cap. It does so by putting the VMs exceeding its share into the lower priority of OVER. However, experiments show that the credit scheduler has high error rate of overallocating CPU shares.



#### KVM/Linux CFS scheduler

KVM, which is essentially a Linux kernel module, relies on the Linux Com- pletely Fair Scheduler(CFS) scheduler. Similar to Xen credit scheduler, CFS is also a proportional share based scheduler, and allocates CPU timeslice to guest VMs as well as other host processes. Below are the summary:

- Administrators can specify the VM’s(more precisely, its corresponding process group’s in the KVM host) nice value to change its weight which is also relative. CPUs are allocated proportional to the VM’s weighted share. CFS allows the administrator to set a resource limit (e.g., by the ulimit command) for a certain process group.

- CFS tracks the VM CPU usage with a [virtual runtime](https://www.kernel.org/doc/Documentation/scheduler/sched-design-CFS.txt) data structure. The granularity is by nanosecond.

- CFS has an optional functionality to [enforce resource limits](https://www.kernel.org/doc/Documentation/scheduler/sched-bwc.txt). The limit is enforced on process groups. CFS throttles the processes in the groups that exceed their limits. To the best of our knowledge, there is no existing study to evaluate the effectiveness of this enforcement mechanism in KVM/Linux environment.

### Memory
A common practice of memory management in Xen is static partitioning. The VMM assigned a fixed amount of physical memory for a guest VM and cannot change the assignment without rebooting the VM. VMs have the privilege to read the hardware page tables. 

However, the VMM tracks all guest updates to the page tables and makes sure that those updates do not violate the rules, including 1) no mapping other VM page frames, 2) no illegal change of privilege on the special pages(e.g., page table pages). Such functional isolation guarantees that the VM memory usage will not be interfered by other VMs. Also, this is a constrained model such that a VM’s idle memory cannot be used by other active VMs.

Similar to Xen, KVM’s memory is often static partitioned. Although it is possible to overcommit the host memory with kernel paging/swapping, it is best practice to ensure a guest VM assigned memory less than the available physical memory. Overcommitting memory is [not recommended](https://access.redhat.com/documentation/en-US/Red Hat Enterprise Linux/6/html/Virtualization Administrati Virtualization-Tips and tricks-Overcommitting with KVM.html).

Note that there is an on-going effort to implement dynamic memory control in Xen Cloud Platform(XCP). Such a feature is implemented by ballooning [5]. In this model, each VM is guaranteed a minimum amount (i.e., reserve) of memory. A VMM can assign more host memory to a VM than its reserve by ”deflating” the balloon, when there is available memory. When the host is under pressure, it ”inflates” the balloon to reclaim the VM memory. 

Developers are anticipating to incorporate the VMM swapping/paging and page sharing features. Although the VMWare ESX paper [5] shows that ballooning only has small impact on the application performance, we do not find the existing study that quantify the performance interference due to ballooning and evaluate the resource limit enforcement mechanisms in XCP or KVM.

### IO

In order to reduce the VMM complexity, the risk of driver failure or development work, both Xen and KVM delegates the communication with hardware to Linux drivers. 

Xen uses a para-virtualized design where the VMM emu- lates a simple device abstraction for the guest VMs. It requires each guest OS to install a light-weight device driver that acts as a ”front-end”. Administrators can assign the physical device to a designated driver domain. The driver domain installs the actual device driver and acts like a ”back-end” to handle the requests from ”front-end”. The communication between driver and other domains is via an efficient event delivery mechanism. 

In KVM, the design is similar but simpler: No driver domain is needed; Since KVM is essentially a Linux kernel module, the VM interfacing with hardware is through inter- module communication. Note that KVM also supports a para-virtualized [virtio](http://www.linux-kvm.org/page/Virtio) driver to achieve better IO performance.

With such a driver delegation design, the IO scheduling policy and control mechanism is subject to the driver and the control domain. As discussed next, both Xen and KVM face the inaccurate resource accounting problems due to this design.

With such a driver delegation design, the IO scheduling policy and control mechanism is subject to the driver and the control domain. As discussed next, both Xen and KVM face the inaccurate resource accounting problems due to this design.

### IO Task Boosting

Since virtualization platforms are designed to host general purpose systems, users expect the systems to achieve good throughput or high utilization and low response latency. These two goals are usually conflicting. 

To achieve low latency, the existing CPU scheduler is adapted to prioritize the newly waken IO bound tasks over CPU bound ones. For example, Linux CFS scheduler does so by tracking the task time-slice usage. A IO bound VMs that spend the majority of time waiting for IO often does not use up its time-slice. The priority of IO bound VMs will be boosted once it wakes up and preempt the long-running CPU bound VM. 

Similar to Linux/KVM, Xen adds a BOOST mechanism. BOOST is a VM state that is analogous to UNDER and OVER states. A VM is boosted when the scheduler detects that there is a virtual interrupt for that VM (perhaps due to a pending IO). 

However, both mechanisms in Xen and KVM are not always efficient and the schedulers sometimes fail to boost IO bound task priority, leading to long scheduling delay.

## Performance Isolation Issues with Xen and KVM

### Incorrect resource accounting

In both Xen and Linux/KVM, the functionality of interfacing with hardware is delegated to third-party drivers. The issue with this delegation design is incorrect performance accounting. 

The root cause is that since the third party driver is outside of VMM and guest OS, existing driver implementations lacks of the necessary information to do the correct resource accounting. As a result, VMM resource scheduler neglect to account the work done on behalf of individual VMs. 

Note that this problem is not unique in virtualization environment and earlier studies have discussions in preventing QoS crosstalk in OS [6] and network [7]. For brevity, we focus the issue on the specific contexts of Xen and KVM.

Study [8] shows that Xen has this type of problem in network IO. The paper shows that the CPU usage of an IO task from a VM is divided into two parts, one inside the VM and the other in the driver domain. Xen CPU scheduler does not charge the packet processing work done on behalf of that VM, and it leads to that some IO-intensive VMs can negatively impact the resource usage by others. As a result, even though we specify a per-VM CPU limit, Xen scheduler does a poor job in enforcing the limit. 

To address this issue, one way is to instrument the driver software to account the resource usage and thus requires to modify all network driver code, which is a difficulty task. Instead, the paper takes an indirect approach. The authors establish that the number of packet sent/received per VM is proportional to the CPU usage on behalf of guests. Therefore, they can derive the CPU usage breakdown by observing the packets in the ”netback” driver. 

The authors also adapt the Xen Simple Earliest Deadline Scheduler(SEDF) scheduler to incorporate the extra CPU accounting information from the driver domain and makes more fair scheduling decisions. Also, the paper proposes a mecha- nism to enforce the resource limit, that is, they place the control point in the ”netback” driver and throttle the network traffic from/to a domain reaching its resource limit.

A 2015-year study [9] also identifies the similar problem in Linux file system. In Linux, all IO requests are streamed to the lowest block-level scheduler which sits beneath the file system and above device. The block-level scheduler decides which IO to be dispatched and when. 

The problem with this design is that block level schedulers lack of the important information about which IO request maps to which process. As as result, they fail to account the resource usage. One consequence is that the SCS token bucket scheduler, which has a mechanism to throttle processes to enforce resource limits, does a poor job in isolating IO performance because it does not know when is the correct time to throttle. 

The paper proposes a split-level scheduling design, in which the scheduler is constructed across the system-call, page-cache, and block layers. The novel schedulers take advantage of the high- and low-level knowledge of the IO requests to perform accurate accounting.

### Inaccurate CPU allocation

Previous studies [10, 11] have shown that Xen credit and SEDF schedulers have high error rate when allocating CPUs. More specifically, Xen credit scheduler tends to over-allocating CPU share to guest VMs. Since we do not find similar studies in KVM, this is likely an Xen-specific implementation issue. The 2015 paper [11] proposes a more accurate scheduler named PRGS that can limit the allocation error within a reasonable range.

### IO tail latency
As discussed previously, systems are expected to prioritize IO bound workloads to achieve low response latency. However, the scheduling entity in Xen and KVM is either a VM or a process group, which is coarse-grained. The hypervisor scheduler has limited knowledge of the tasks in guest VMs. When making the scheduling decision, while it works well in most cases, it sometimes fails to boost the priority of IO bound tasks. Existing studies have confirmed the long IO latency issues in both Xen and KVM. Below we discuss two scenarios where this problem may occur.

#### Intra-VM mix of IO and CPU bound workloads
Both Xen and KVM suffer from occasional long IO response when we mix the IO and CPU bound tasks inside a single VM. 

In Xen, while the BOOST mechanism works well for pure IO bound VMs, but it fails in mixed workloads because the deficiency of credit scheduler itself [12]. More precisely, once the VM gets BOOSTed to process I/O requests. However, because of the colocated CPU bound workload, the VM soon exhausts all its credits too early and be evaluated by the scheduler as OVER state which will disable the BOOST mechanism. Further IOs need to wait for the rest of this scheduling cycle, resulting high I/O latencies. 

Similar issues exist in KVM as well [13]. Therefore, it is generally not recommended to run mixed workloads 9 in one single VM.

A naive solution to this issue is to boost all IO bound VMs regardless of its credit consumed, but this leads to unfairness in CPU usage. 

An earlier paper [14] proposes partial boosting as a solution to this problem. With this technique, the VMM can infer whether a VM is scheduling a IO bound task. If true, the VM is boosted for a short period and preempt the current CPU bound VM. The VMM revokes the CPU when it infers that the VM is not executing on a IO bound tasks any more. 

[12] argues that partial boosting requires complex configurations and may hurt the CPU allocation fairness in the short term. Instead, [12] proposes another approach which reduces the timeslice size for latency sensitive VMs and increase the scheduling frequency so as to achieve both fairness and low IO latency.

#### Inter-VM mix of IO and CPU bound workloads
This scenario is more subtle than the previous. Study [15] designs a controlled experiment in Xen and shows that, a VM running IO bound tasks alone, without any CPU intensive tasks, can still suffer from occasional 2-4X latency if there exists co-located CPU intensive VMs that share physical cores with the victim VM. 

Somewhat surprisingly, sharing CPU cores alone does not cause the latency unless the following two conditions are met: 1) the CPU intensive VMs outnumber the available physical cores and 2) the CPU inten- sive VMs are not consuming 100% CPU. The authors of [15] attribute this phenomenon to the Xen BOOST mechanism: Though rare, a CPU (some- what) intensive VM can also get BOOSTed by using certain functions, e.g., sleep(). 

As a result, this VM ends up in the same priority queue with other IO bound VMs, and continue to monopolize the CPU core. This is against the design rationale of the BOOST mechanism.

Here, the ”occasional latency” is actually tail latency, a rare phenomenon that occurs in the 99.9 percentile (or higher) of the latency distribution. Nev- ertheless, the possible occurrence of tail latencies add to the unpredictable performance of multi-tenant clouds (e.g., Amazon EC2) where a tenant has no control over what others are doing. 

To make things worse, the tail latency can be amplified by scale and impact a wide range of users [16]. [15] proposes to mitigate this issue by pro-actively detecting and avoiding interference VMs through optimized VM placement.


## Conclusion
This article examines the resource models of Xen and KVM, and summarizes the existing work on VM performance isolation topics. We find that Xen and KVM share many similarities in resource model and performance isolation problems. 

At the center of the problems are VMM CPU schedulers. In particular, the known VMM schedulers have the problems of inaccurate CPU usage accounting and CPU share mis-allocation, which can to unfairness among guest VMs. 

Also, in order to increase responsiveness, both Xen and KVM schedulers have mechanisms to boost IO bound tasks. However, due to the coarse-grained scheduling entity, VMs running in both platforms are likely to suffer from tail latency in IOs.

While existing studies have addressed some of the issues, we identify the following future work directions:

- Evaluate the performance isolation aspects in the newest version of Xen and KVM, including the effectiveness of resource limit enforcement mechanisms in KVM, performance guarantees with dynamic memory control enabled.

- An exploitative study on how to devise a strategy for the customer VMs that experience unexpected tail latency. This will be useful for public cloud tenants.

- A split-level scheduler design in the VMM similar to [9] that can achieve more accurate resource accounting.

- A ”smarter” VM placement and migration management that incorpo- rates latency goals and energy saving policies.

## References

[1] Paul Barham, Boris Dragovic, Keir Fraser, Steven Hand, Tim Harris, Alex Ho, Rolf Neugebauer, Ian Pratt, and Andrew Warfield. Xen and the art of virtualization. ACM SIGOPS Operating Systems Review, 37(5):164–177, 2003.

[2] Avi Kivity, Yaniv Kamay, Dor Laor, Uri Lublin, and Anthony Liguori. kvm: the linux virtual machine monitor. In Proceedings of the Linux Symposium, volume 1, pages 225–230, 2007.

[3] Forbes Guthrie, Scott Lowe, and Kendrick Coleman. VMware vSphere design. John Wiley & Sons, 2013.

[4] Armando Fox, Rean Griffith, Anthony Joseph, Randy Katz, Andrew Konwinski, Gunho Lee, David Patterson, Ariel Rabkin, and Ion Stoica. Above the clouds: A berkeley view of cloud computing. Dept. Electrical Eng. and Comput. Sciences, University of California, Berkeley, Rep. UCB/EECS, 28:13, 2009.

[5] Carl A Waldspurger. Memory resource management in vmware esx server. ACM SIGOPS Operating Systems Review, 36(SI):181–194, 2002.

[6] Ian M Leslie, Derek McAuley, Richard Black, Timothy Roscoe, Paul Barham, David Evers, Robin Fairbairns, and Eoin Hyden. The design and implementation of an operating system to support distributed mul- timedia applications. Selected Areas in Communications, IEEE Journal on, 14(7):1280–1297, 1996.

[7] David L Tennenhouse. Layered multiplexing considered harmful. In IFIP Workshop on Protocols for High-Speed Networks. Elsevier, 1989.

[8] Diwaker Gupta, Ludmila Cherkasova, Rob Gardner, and Amin Vah- dat. Enforcing performance isolation across virtual machines in xen. In Middleware 2006, pages 342–362. Springer, 2006.

[9] Suli Yang, Tyler Harter, Nishant Agrawal, Salini Selvaraj Kowsalya, Anand Krishnamurthy, Samer Al-Kiswany, Rini T Kaushik, Andrea C Arpaci-Dusseau, and Remzi H Arpaci-Dusseau. Split-level i/o schedul- ing. In Proceedings of the 25th Symposium on Operating Systems Prin- ciples, pages 474–489. ACM, 2015.

[10] Ludmila Cherkasova, Diwaker Gupta, and Amin Vahdat. Comparison of the three cpu schedulers in xen. SIGMETRICS Performance Evaluation Review, 35(2):42–51, 2007.

[11] Jian Li, David SL Wei, et al. Accurate cpu proportional share and predictable i/o responsiveness for virtual machine monitor: A case study in xen.

[12] Cong Xu, Sahan Gamage, Pawan N Rao, Ardalan Kangarlou, Ra- mana Rao Kompella, and Dongyan Xu. vslicer: latency-aware virtual machine scheduling via differentiated-frequency cpu slicing. In Proceed- ings of the 21st international symposium on High-Performance Parallel and Distributed Computing, pages 3–14. ACM, 2012.

[13] Hyotaek Shim and Sung-Min Lee. Cfs-v: I/o demand-driven vm sched- uler in kvm. In Proceedings of the KVM Forum, 2014.

[14] Hwanju Kim, Hyeontaek Lim, Jinkyu Jeong, Heeseung Jo, and Joonwon Lee. Task-aware virtual machine scheduling for i/o performance. In Pro- ceedings of the 2009 ACM SIGPLAN/SIGOPS international conference on Virtual execution environments, pages 101–110. ACM, 2009.

[15] Yunjing Xu, Zachary Musgrave, Brian Noble, and Michael Bailey. Bob- tail: Avoiding long tails in the cloud. In NSDI, pages 329–341, 2013.

[16] Jeffrey Dean and Luiz Andr ́e Barroso. The tail at scale. Communications of the ACM, 56(2):74–80, 2013.
