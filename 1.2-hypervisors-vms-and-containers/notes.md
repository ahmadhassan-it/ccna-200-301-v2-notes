# 1.2 Hypervisors, Virtual Machines and Containers

**CCNA 200-301 v2.0, topic 1.2:** Describe the role and function of hypervisors, virtual machines, and containers.

Study sources: Jeremy's IT Lab (Day 54, virtualization and containers) and my own tests. Notes checked for technical mistakes with Claude. I can still be wrong, so verify important facts in Jeremy's videos or Cisco documents.

---

## Contents

1. [The problem: one server, one job](#1-the-problem-one-server-one-job)
2. [What a hypervisor does](#2-what-a-hypervisor-does)
3. [Type 1 vs Type 2](#3-type-1-vs-type-2)
4. [How a VM connects to the network](#4-how-a-vm-connects-to-the-network)
5. [Containers](#5-containers)
6. [VM vs container](#6-vm-vs-container)
7. [Common mistakes](#7-common-mistakes)
8. [Exam angle and job angle](#8-exam-angle-and-job-angle)
9. [Must memorize](#9-must-memorize)
10. [Word list](#10-word-list)
11. [My mistake log](#11-my-mistake-log)

---

## 1. The problem: one server, one job

Old way: one job = one physical server (web server, mail server, database server).

- Each server was busy only a small part of the time.
- The company still paid for the hardware, power, cooling and space of **every** server, even when it was mostly idle.

**Idea:** one real computer can act like many separate computers, using software.

Real-life comparison: one apartment building, many flats. Each family has its own door and kitchen. The building manager (the hypervisor) decides how much water and electricity each flat gets.
*Where it breaks:* a flat is made of real walls. A VM is software, so it can be copied, paused or moved to another server.

---

## 2. What a hypervisor does

A **hypervisor** is software that creates and runs virtual machines (VMs).

```
 +----------+  +----------+  +----------+
 |   VM 1   |  |   VM 2   |  |   VM 3   |
 | own OS   |  | own OS   |  | own OS   |
 +----------+  +----------+  +----------+
 +--------------------------------------+
 |             HYPERVISOR               |
 +--------------------------------------+
 +--------------------------------------+
 | Hardware: CPU, RAM, disk, NIC        |
 +--------------------------------------+
```

- It shares the server's CPU, memory, disk and network between the VMs.
- It keeps the VMs **separate**. If VM 2 stops, VM 1 and VM 3 keep running.
- Each VM has its own full operating system and thinks it is a real computer.
- A VM is software. Its disk and settings are stored as files, so it can be copied or moved to another server that runs a compatible hypervisor.

---

## 3. Type 1 vs Type 2

One question decides the type: **what is under the hypervisor?**

```
   TYPE 1 (bare metal)           TYPE 2 (hosted)

 [VM1] [VM2] [VM3]             [VM1] [VM2]
 +----------------+            +----------------+
 |  HYPERVISOR    |            |  HYPERVISOR    |
 +----------------+            |  (an app)      |
 +----------------+            +----------------+
 |   HARDWARE     |            |  HOST OS       |
 +----------------+            +----------------+
                               |   HARDWARE     |
                               +----------------+
```

| | Type 1 | Type 2 |
|---|---|---|
| Hypervisor runs on | the hardware (bare metal) | a host OS |
| Examples | VMware ESXi | VirtualBox, VMware Workstation |
| Used in | data centres | laptops, labs, practice |
| Host OS goes down | no host OS to go down | hypervisor and all VMs go down |

**In both types the VMs run on the hypervisor.** The difference is what the **hypervisor** runs on.

Example: Windows laptop, VirtualBox, 3 Linux VMs. Windows restarts for an update, so all 3 VMs stop. VirtualBox is just an app on Windows.

Type 1 does not make a server crash-proof. The hypervisor can need updates or restarts, and hardware can fail. It only removes the host OS as one weak point.

---

## 4. How a VM connects to the network

```
 VM1 --vNIC--[vSwitch X]--[physical NIC]--[physical switch]--> network
 VM2 --vNIC--[vSwitch X]

 VM3 --vNIC--[vSwitch Y]      (no physical NIC = no way out)
 VM4 --vNIC--[vSwitch Y]
```

- **vNIC:** a virtual network card. Every VM gets one.
- **vSwitch:** a switch made by software, inside the hypervisor.
- **Uplink:** the link from a vSwitch to a physical NIC.

Rules:
- A vSwitch with an uplink: its VMs can reach the real network.
- A vSwitch with **no** uplink: its VMs can talk only to each other.
- Two different vSwitches do not talk to each other (unless something routes between them).

Troubleshooting pattern: VMs on the same vSwitch can ping each other but cannot reach the office network, while VMs on another vSwitch work fine. Look for a **missing uplink** on that vSwitch. The physical NIC is working, because the other VMs use it.

---

## 5. Containers

Problem: every VM carries a full OS. That is heavy. Often we only want to run one small app.

**A container packs one app with the files and libraries it needs, and shares the host's OS kernel.**

```
 VMs                           Containers

 [App1+libs] [App2+libs]       [App1+libs] [App2+libs]
 [Guest OS ] [Guest OS ]       [  Container engine   ]
 [    Hypervisor       ]       [      Host OS        ]
 [    Hardware         ]       [      Hardware       ]
```

- No OS of its own, so a container is smaller and starts faster (in seconds).
- Docker is one popular container tool (just know the name for now).
- A container is **not** a very small VM. It has no OS of its own.

---

## 6. VM vs container

| | VM | Container |
|---|---|---|
| Own OS | yes (full guest OS) | no, shares the host kernel |
| Size | bigger | smaller |
| Start time | slower | faster |
| Separation from others | stronger (own kernel) | weaker (shared kernel) |
| Good for | different OSs, strong separation | many small apps, fast start |

Because containers share the kernel, without limits one container can use too much CPU and slow the others. Containers are not isolated as strongly as VMs.

---

## 7. Common mistakes

- "The VM and the hypervisor are the same thing." No. The hypervisor creates and manages the VMs.
- "Type 2 is a different kind of VM." No. Type 1 and Type 2 describe the **hypervisor**.
- "A container is a small VM." No. A container has no OS of its own.
- "A vSwitch with one VM has no network." No. What matters is whether the vSwitch has an uplink.
- "VMs run directly on the hardware." No. The **hypervisor** runs on the hardware (Type 1). The VMs run on the hypervisor.

---

## 8. Exam angle and job angle

**Exam angle:** the topic says *describe*. Expect questions like: what is a hypervisor, Type 1 vs Type 2, VM vs container, which one for strong separation or fast start. Watch the word **"directly"** and what the question says is under the hypervisor.

**Job angle:** many servers in data centres and cloud run as VMs, and containers are common in DevOps work (Docker and Kubernetes come later in my plan).

---

## 9. Must memorize

- Hypervisor = software that creates and runs VMs.
- Type 1: hypervisor runs directly on the **hardware** (data centres).
- Type 2: hypervisor runs on a **host OS** (laptops). Host OS down = VMs down.
- VM: own OS, own kernel, bigger, slower start, stronger separation.
- Container: shares the host kernel, smaller, faster start, weaker separation.
- VM network path: vNIC, vSwitch, uplink, physical NIC, physical switch.
- vSwitch with no uplink = no outside network.
- A VM is software, so it can be copied or moved.

**Memory hook:** VM = a house with its own kitchen. Container = a room in a shared flat.

---

## 10. Word list

| Word | Simple meaning |
|---|---|
| Hypervisor | Software that creates and runs VMs |
| VM (virtual machine) | A computer made by software inside a real computer |
| Host OS | The normal OS under a Type 2 hypervisor |
| Guest OS | The OS inside a VM |
| Bare metal | Installed straight on the hardware, no OS under it |
| Type 1 | Hypervisor on the hardware |
| Type 2 | Hypervisor on a host OS |
| NIC | Network interface card, connects a machine to the network |
| vNIC | Virtual network card inside a VM |
| vSwitch | Switch made by software inside the hypervisor |
| Uplink | Link from a vSwitch to a physical NIC |
| Container | An app packed with its files, sharing the host kernel |
| Container engine | Software that runs containers (Docker is an example) |
| Kernel | Core part of an OS that talks to the hardware |
| Isolation | How strongly one thing is kept apart from another |
| Library | Ready-made code that an app uses |
| Linux / Ubuntu | Linux is a free OS. Ubuntu is one popular version of it |

---

## 11. My mistake log

- Said "Type 1 VMs run directly on hardware". The **hypervisor** runs on the hardware, the VMs run on the hypervisor.
- Wrote "Type 1 runs on hypervisor". In both types the VMs run on the hypervisor. The difference is what the hypervisor runs on.
- Thought a vSwitch with one VM has no network. The truth: it depends on the uplink.
- Gave a result ("containers are efficient and faster") instead of the reason (they share the host OS, so no OS per app).
- Said Type 1 would not fix a host-OS restart problem. The truth: Type 1 has no host OS, so that cause is gone.
- Used "crash" for VMs that stop. Better: "go down" or "stop".

---

**Lab:** none for this topic. Packet Tracer cannot run hypervisors, VMs or containers.
