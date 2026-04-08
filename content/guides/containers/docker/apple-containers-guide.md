---
title: Apple Containers on macOS – Early Setup and First Container
description: Try Apple’s native container runtime on macOS and run your first container from the terminal.
date: 2026-04-08
tags: [apple, containers, macos, docker]
---

{{< callout type="warning" >}}
**Experimental feature.** Apple Containers is still under development and may change or break between updates. Use only in test environments.
{{< /callout >}}

Apple is working on native container support in macOS. This is an early implementation and not yet a full replacement for Docker or OrbStack.

This guide shows how to test it and run a basic container.

---

## Why Apple Containers?

- Native macOS integration  
- Potentially lower overhead than traditional solutions  
- No third-party runtime required  
- Interesting for future development  

---

## Requirements

- macOS (latest version recommended)  
- Apple Silicon recommended  
- Xcode Command Line Tools  

Install CLI tools if needed:

```bash
xcode-select --install
```

---

## Step 1 – Check if container support exists

Apple’s container runtime is still evolving. Try:

```bash
container --help
```

If available, you’ll see command options.

If not:
- your macOS version may not support it yet

---

## Step 2 – Pull an image

Example:

```bash
container pull alpine
```

---

## Step 3 – Run a container

```bash
container run alpine echo "Hello from Apple Containers"
```

---

## Step 4 – Interactive container

```bash
container run -it alpine sh
```

Inside:

```bash
apk add curl
```

Exit:

```bash
exit
```

---

## Notes

- Not all Docker features are supported  
- CLI may change between macOS versions  
- No full Docker compatibility yet  
- Best used for testing and exploration  

---

## When to use this

- Testing Apple’s ecosystem  
- Learning container fundamentals  
- Experimenting with future tooling  

For production or daily use, OrbStack or Docker is still recommended.

---

## Related Links

- https://developer.apple.com/
- https://docs.docker.com/
