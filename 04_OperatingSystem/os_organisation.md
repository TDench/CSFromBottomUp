# 作業系統架構

作業系統的架構大概長的像下面張圖這樣

<figure><img src="../.gitbook/assets/image.png" alt=""><figcaption></figcaption></figure>

## 作業系統核心 (Kernel)

所謂的作業系統核心，以下簡稱核心，就是指作業系統。就跟上面那張圖顯示的一樣，核心會跟直接或是特過驅動跟硬體溝通。

核心會抽象化硬體，讓程式可以運行。驅動抽象化硬體，讓核心可以運行。舉例來說，我們有很多不同類型的顯示卡，每張顯示卡都有自己硬體上的特色。只要核心輸出一個API，知道硬體規格的人就可以寫驅動的程式來實現這個API，讓核心可以藉由這個API存取各種不同的硬體。

核心通常被稱為特權(privileged)。後面會學到，硬體在多工處理跟系統安全性有很重要的角色，但是這些規則不適用於核心。我們知道核心需要處理當掉的程式(剛剛有提到作業系統的功能就是分配多個任務的資源，但不會保證這個程式的運行狀態，也就是當掉是程式會發生的憾事，作業系統要處能處理)。但是如果作業系統當掉了，那整台電腦都會當掉。同樣的，如果使用者的行程 (process)可以把自己的安全級別提升到kernel等級的權限的話，那他就控制了整台電腦了。

{% hint style="info" %}
這段應該是在說核心有特權，安全性跟例外處理是跟一般process完全不同的。
{% endhint %}

單一核心(**Monolithic)  (Microkernels)**

One debate that is often comes up surrounding operating systems is whether the kernel should be a _microkernel_ or _monolithic_.

The monolithic approach is the most common, as taken by most common Unixes (such as Linux). In this model the core privileged kernel is large, containing hardware drivers, file system accesses controls, permissions checking and services such as Network File System (NFS).

Since the kernel is always privileged, if any part of it crashes the whole system has the potential to come to a halt. If one driver has a bug, it can overwrite any memory in the system with no problems, ultimately causing the system to crash.

A microkernel architecture tries to minimise this possibility by making the privileged part of the kernel as small as possible. This means that most of the system runs as unprivileged programs, limiting the harm that any one crashing component can influence. For example, drivers for hardware can run in separate processes, so if one goes astray it can not overwrite any memory but that allocated to it.

Whilst this sounds like the most obvious idea, the problem comes back two main issues

Whilst this sounds like the most obvious idea, the problem comes back two main issues

1. Performance is decreased. Talking between many different components can decrease performance.
2. It is slightly more difficult for the programmer.

Both of these criticisms come because to keep separation between components most microkernels are implemented with a _message passing_ based system, commonly referred to as _inter-process communication_ or IPC. Communicating between individual components happens via discrete messages which must be bundled up, sent to the other component, unbundled, operated upon, re-bundled up and sent back, and then unbundled again to get the result.

This is a lot of steps for what might be a fairly simple request from a foreign component. Obviously one request might make the other component do more requests of even more components, and the problem can multiply. Slow message passing implementations were largely responsible for the poor performance of early microkernel systems, and the concepts of passing messages are slightly harder for programmers to program for. The enhanced protection from having components run separately was not sufficient to overcome these hurdles in early microkernel systems, so they fell out of fashion.

In a monolithic kernel, calls between components are simple function calls, as all programmers are familiar with.

There is no definitive answer as to which is the best organisation, and it has started many arguments in both academic and non-academic circles. Hopefully as you learn more about operating systems you will be able to make up your own mind!

**2.1.1.1 Modules**

The Linux kernel implements a module system, where drivers can be loaded into the running kernel "on the fly" as they are required. This is good in that drivers, which make up a large part of operating system code, are not loaded for devices that are not present in the system. Someone who wants to make the most generic kernel possible (i.e. runs on lots of different hardware, such as RedHat or Debian) can include most drivers as modules which are only loaded if the system it is running on has the hardware available.

However, the modules are loaded directly in the privileged kernel and operate at the same privilege level as the rest of the kernel, so the system is still considered a monolithic kernel.

**2.1.2 Virtualisation**

Closely related to kernel is the concept of virtualisation of hardware. Modern computers are very powerful, and often it is useful to not think of them as one whole system but split a single physical computer up into separate "virtual" machines. Each of these virtual machines looks for all intents and purposes as a completely separate machine, although physically they are all in the same box, in the same place.

<figure><img src="../.gitbook/assets/image (1).png" alt=""><figcaption></figcaption></figure>

\
This can be organised in many different ways. In the simplest case, a small _virtual machine monitor_ can run directly on the hardware and provide an interface to the guest operating systems running on top. This VMM is often often called a hypervisor (from the word "supervisor")[1](https://www.bottomupcs.com/ch04s02.html#organisation\_of\_os\_s1\_s2\_para2\_footnote1-fnote). In fact, the operating system on top may have no idea that the hypervisor is even there at all, as the hypervisor presents what appears to be a complete system. It intercepts operations between the guest operating system and hardware and only presents a subset of the system resources to each.

This is often used on large machines (with many CPUs and much RAM) to implement _partitioning_. This means the machine can be split up into smaller virtual machines. Often you can allocate more resources to running systems on the fly, as requirements dictate. The hypervisors on many large IBM machines are actually quite complicated affairs, with many millions of lines of code. It provides a multitude of system management services.

Another option is to have the operating system be aware of the underlying hypervisor, and request system resources through it. This is sometimes referred to as _paravirtualisation_ due to its halfway nature. This is similar to the way early versions of the Xen system work and is a compromise solution. It hopefully provides better performance since the operating system is explicitly asking for system resources from the hypervisor when required, rather than the hypervisor having to work things out dynamically.

Finally, you may have a situation where an application running on top of the existing operating system presents a virtualised system (including CPU, memory, BIOS, disk, etc) which a plain operating system can run on. The application converts the requests to hardware through to the underlying hardware via the existing operating system. This is similar to how VMWare works. This approach has many overheads, as the application process has to emulate an entire system and convert everything to requests from the underlying operating system. However, this lets you emulate an entirely different architecture all together, as you can dynamically translate the instructions from one processor type to another (as the Rosetta system does with Apple software which moved from the PowerPC processor to Intel based processors).

Performance is major concern when using any of these virtualisation techniques, as what were once fast operations directly on hardware need to make their way through layers of abstraction.

Intel have discussed hardware support for virtualisation soon to be coming in their latest processors. These extensions work by raising a special exception for operations that might require the intervention of a virtual machine monitor. Thus the processor looks the same as a non-virtualised processor to the application running on it, but when that application makes requests for resources that might be shared between other guest operating systems, the virtual machine monitor can be invoked.

This provides superior performance because the virtual machine monitor does not need to monitor every operation to see if it is safe, but can wait until the processor notifies that something _unsafe_ has happened.

**2.1.2.1 Covert Channels**

This is a digression, but an interesting security flaw relating to virtualised machines. If the partitioning of the system is not static, but rather _dynamic_, there is a potential security issue involved.

In a dynamic system, resources are allocated to the operating systems running on top as required. Thus if one is doing particularly CPU intensive operations whilst the other is waiting on data to come from disks, more of the CPU power will be given to the first task. In a static system, each would get 50% an the unused portion would go to waste.

Dynamic allocation actually opens up a communications channel between the two operating systems. Anywhere that two states can be indicated is sufficient to communicate in binary. Imagine both systems are extremely secure, and no information should be able to pass between one and the other, ever. Two people with access could collude to pass information between themselves by writing two programs that try to take large amounts of resources at the same time.

When one takes a large amount of memory there is less available for the other. If both keep track of the maximum allocations, a bit of information can be transferred. Say they make a pact to check every second if they can allocate this large amount of memory. If the target can, that is considered binary 0, and if it can not (the other machine has all the memory), that is considered binary 1. A data rate of one bit per second is not astounding, but information is flowing.

This is called a _covert channel_, and whilst admittedly far-fetched there have been examples of security breaches from such mechanisms. It just goes to show that the life of a systems programmer is never simple!

#### 2.2 Userspace

We call the theoretical place where programs are run by the user _userspace_. Each program runs in userspace, talking to the kernel through _system calls_ (discussed below).

As previously discussed, userspace is _unprivileged_. User programs can only do a limited range of things, and should never be able to crash other programs, even if they crash themselves.





