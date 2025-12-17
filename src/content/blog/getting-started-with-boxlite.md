---
draft: false
title: "Getting Started with BoxLite: Your First Hardware-Isolated Sandbox"
snippet: "A step-by-step tutorial on creating your first BoxLite sandbox. Learn how to run Python code safely in hardware-isolated micro-VMs in just a few minutes."
image: {
    src: "https://images.unsplash.com/photo-1629654297299-c8506221ca97?&fit=crop&w=430&h=240",
    alt: "Getting started with BoxLite sandboxing"
}
publishDate: "2024-12-10 10:00"
category: "Tutorials"
author: "BoxLite Team"
tags: [tutorial, python, getting-started, sandbox]
---

**TL;DR:** Install BoxLite with `pip install boxlite`, then create a sandbox with `async with boxlite.SimpleBox("python:slim") as box`. Your code runs in a hardware-isolated micro-VM with its own kernel. No daemon, no root, no complex setup.

## Prerequisites

Before starting, make sure you have:

**macOS (Apple Silicon):**
- macOS 12 or later
- M1/M2/M3/M4 chip
- Python 3.10+

**Linux:**
- KVM enabled (`/dev/kvm` accessible)
- User in the `kvm` group: `sudo usermod -aG kvm $USER`
- Python 3.10+

## Step 1: Install BoxLite

```bash
pip install boxlite
```

That's it. No daemon to start, no Docker to install, no root access needed.

## Step 2: Your first sandbox

Create a file called `hello_sandbox.py`:

```python
import asyncio
import boxlite

async def main():
    # Create a sandbox using the official Python image
    async with boxlite.SimpleBox("python:slim") as box:
        # Run a command inside the sandbox
        result = await box.exec(
            "python", "-c",
            "print('Hello from inside the sandbox!')"
        )
        print(result.stdout)

asyncio.run(main())
```

Run it:

```bash
python hello_sandbox.py
```

The first run will pull the `python:slim` image (this takes a moment). Subsequent runs are fast thanks to layer caching.

## Step 3: Understanding what happened

When you ran that code:

1. BoxLite pulled the OCI image from Docker Hub
2. Created a micro-VM with its own Linux kernel
3. Started the container inside the VM
4. Executed your Python command
5. Captured stdout and returned it
6. Cleaned up the VM

The key point: your code ran in a **hardware-isolated VM**, not just a container. Even if the code tried to exploit a container escape vulnerability, it would still be trapped inside the VM.

## Step 4: Running more complex code

Let's run multi-line Python code:

```python
import asyncio
import boxlite

code = '''
import os
import sys

print(f"Python version: {sys.version}")
print(f"Current directory: {os.getcwd()}")
print(f"User: {os.getenv('USER', 'unknown')}")

# This runs inside the sandbox - can't affect your host
with open('/tmp/sandbox_test.txt', 'w') as f:
    f.write('Data stays inside the sandbox')

print("File written successfully!")
'''

async def main():
    async with boxlite.SimpleBox("python:slim") as box:
        result = await box.exec("python", "-c", code)
        print("=== Output ===")
        print(result.stdout)
        if result.stderr:
            print("=== Errors ===")
            print(result.stderr)
        print(f"=== Exit code: {result.exit_code} ===")

asyncio.run(main())
```

## Step 5: Using CodeBox for Python specifically

BoxLite provides a specialized `CodeBox` API for Python execution:

```python
import asyncio
import boxlite

async def main():
    async with boxlite.CodeBox() as box:
        # CodeBox handles Python execution details
        result = await box.run('''
import numpy as np

# NumPy is pre-installed in CodeBox
arr = np.array([1, 2, 3, 4, 5])
print(f"Array: {arr}")
print(f"Mean: {arr.mean()}")
print(f"Sum: {arr.sum()}")
''')
        print(result.stdout)

asyncio.run(main())
```

`CodeBox` is optimized for Python code execution with:
- Pre-configured Python environment
- Common packages pre-installed
- Automatic error handling

## Step 6: Adding resource limits

Protect against runaway code:

```python
async with boxlite.SimpleBox(
    "python:slim",
    memory_mb=256,        # Limit memory to 256MB
    cpus=1,               # Limit to 1 CPU
) as box:
    result = await box.exec("python", "-c", untrusted_code)
```

## Step 7: Handling errors

Always handle potential failures:

```python
import asyncio
import boxlite

async def run_safely(code: str) -> str:
    try:
        async with boxlite.CodeBox() as box:
            result = await box.run(code)

            if result.exit_code != 0:
                return f"Code failed with exit code {result.exit_code}:\n{result.stderr}"

            return result.stdout

    except Exception as e:
        return f"Sandbox error: {e}"

# Example usage
async def main():
    # This will fail gracefully
    output = await run_safely("import nonexistent_module")
    print(output)

    # This will succeed
    output = await run_safely("print('Hello!')")
    print(output)

asyncio.run(main())
```

## Common issues and solutions

### "Permission denied" on Linux

Make sure you have KVM access:

```bash
# Check if KVM is available
ls -la /dev/kvm

# Add yourself to the kvm group
sudo usermod -aG kvm $USER

# Log out and back in for the change to take effect
```

### First run is slow

The first run downloads the OCI image. Subsequent runs reuse cached layers. You can pre-pull images:

```python
import boxlite

# Pre-pull during setup
boxlite.pull_image("python:slim")
```

### Memory errors

Increase the memory limit:

```python
async with boxlite.SimpleBox("python:slim", memory_mb=1024) as box:
    # 1GB of memory available
    ...
```

## Next steps

Now that you have BoxLite working:

1. **Explore CodeBox**: Optimized for Python code execution
2. **Try BrowserBox**: For browser automation in isolation
3. **Try ComputerBox**: For desktop automation with screenshots
4. **Read the FAQ**: Common questions answered at [/faq](/faq)
5. **Join Discord**: Get help at [discord.gg/TW7jXHyX7s](https://discord.gg/TW7jXHyX7s)

---

*Questions? Check out the [complete examples](https://github.com/boxlite-labs/boxlite-python-examples) or [open an issue](https://github.com/boxlite-labs/boxlite/issues).*
