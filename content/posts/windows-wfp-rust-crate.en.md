---
title: "windows-wfp: A Safe Rust Wrapper for the Windows Kernel Firewall"
date: 2026-03-14T10:00:00+02:00
lastmod: 2026-03-14T10:00:00+02:00
draft: false
weight: 1
tags: ["Rust", "Windows", "WFP", "Firewall", "Security", "Network", "Open Source", "Crate", "FFI", "Kernel"]
categories: ["Security", "Rust", "Open Source"]
author: "lostyzen"
showToc: true
TocOpen: false
hidemeta: false
comments: true
description: "windows-wfp: an open-source Rust crate providing a safe API to interact with the Windows Filtering Platform (WFP), the kernel-level firewall of Windows."
canonicalURL: "https://lostyzen.github.io/en/posts/windows-wfp-rust-crate/"
disableHLJS: false
disableShare: false
searchHidden: false
ShowReadingTime: true
ShowBreadCrumbs: true
ShowPostNavLinks: true
ShowWordCount: true
ShowRssButtonInSectionTermList: true
UseHugoToc: true
cover:
    image: "/images/windows-wfp-rust-crate-og.png"
    alt: "windows-wfp: Rust wrapper for Windows Filtering Platform"
    caption: "A safe Rust wrapper for the Windows kernel-level firewall"
    relative: false
    hidden: false
editPost:
    URL: "https://github.com/lostyzen/lostyzen.github.io/tree/main/content"
    Text: "Suggest changes"
    appendFilePath: true
keywords: ["rust", "windows", "wfp", "windows filtering platform", "firewall", "kernel", "network security", "crate", "open source", "ffi", "safe wrapper", "packet filtering"]
schema:
  type: "BlogPosting"
  datePublished: "2026-03-14T10:00:00+02:00"
  dateModified: "2026-03-14T10:00:00+02:00"
  author: "lostyzen"
  publisher: "DevOps Blog"
images: ["/images/windows-wfp-rust-crate-og.png"]
---

🛡️ **windows-wfp: A Safe Rust Wrapper for the Windows Kernel Firewall**

For the past few months, I've been working on a personal Rust project related to network security on Windows. An ambitious project whose full details I won't reveal just yet — let's just say it requires **interacting directly with the Windows kernel-level firewall**. And when you dive into that world, you inevitably stumble upon the **Windows Filtering Platform (WFP)**.

The problem? Manipulating WFP from Rust is quite the challenge. So I built a wrapper for my own needs and decided to share it as **open-source** on crates.io: **[windows-wfp](https://crates.io/crates/windows-wfp)**.

## 🎯 **What is the Windows Filtering Platform?**

**WFP** (Windows Filtering Platform) is the kernel-level network filtering framework built into Windows since Vista/Server 2008. It's the engine that powers **Windows Firewall** itself, as well as most network security solutions: antivirus, EDR, VPN...

In concrete terms, WFP allows you to:
- **Block or allow** network traffic based on fine-grained rules
- **Filter by application**: allow Firefox but block another app
- **Filter by protocol, port, IP**: with CIDR support for address ranges
- **Monitor network events** in real-time
- **Operate at the kernel level**: before traffic even reaches the application

It's an extremely powerful API, used by professional security tools. But it has one major drawback: it's **designed for C/C++**, and using it from Rust means juggling with low-level, unergonomic abstractions.

## ⚠️ **The Problem: Using WFP Without a Wrapper**

When I started integrating WFP into my Rust project, I quickly faced three major obstacles:

### **1. Raw FFI Everywhere**

**FFI** (Foreign Function Interface) is the mechanism that allows Rust to call Windows C APIs. The problem is that with raw FFI, you lose all the memory safety guarantees that make Rust powerful. You're dealing with raw pointers, C structures, and end up writing `unsafe` code (where you tell the compiler: "trust me, I know what I'm doing" — spoiler: you don't always know 😅).

### **2. Unsecured Windows Handles**

A **handle** in Windows is an opaque identifier the system gives you when you open a resource (a WFP session, a file, a process...). The issue: you must remember to **close each handle manually**. Forget it → resource leak. Use it after closing → undefined behavior. The kind of silent bug that only manifests in production, 3 months later, at 2 AM.

### **3. The NT Kernel Path Trap**

This is probably the most treacherous trap. WFP operates at the kernel level and uses **NT kernel paths** (`\Device\HarddiskVolume3\Windows\System32\notepad.exe`), not the usual DOS paths (`C:\Windows\System32\notepad.exe`).

The result? You add a filter to block `notepad.exe` using its classic path. The WFP API returns **no error** — everything seems to work. But the filter never matches a single packet. Your rules exist, but they're silent. 🙃

I lost considerable time debugging this behavior before understanding the issue. This kind of frustration is precisely what motivated me to encapsulate all this complexity into a clean wrapper.

## 🚀 **windows-wfp: The Idiomatic Rust Approach**

The **windows-wfp** crate encapsulates all this complexity behind a safe, ergonomic, and idiomatic Rust API.

### **Crate Architecture**

```
windows-wfp/
├── src/
│   ├── engine.rs       # WFP session (RAII, auto-cleanup)
│   ├── provider.rs     # Provider registration
│   ├── sublayer.rs     # Sublayer management (priorities)
│   ├── filter.rs       # Builder pattern for filters
│   ├── condition.rs    # Filtering conditions
│   ├── path.rs         # DOS → NT kernel path conversion
│   ├── monitor.rs      # Real-time network monitoring
│   └── lib.rs          # Public entry point
```

### **Design Principles**

✅ **RAII for handles**: Every WFP session, every provider, every filter is wrapped in a Rust type that automatically closes the handle when the object goes out of scope. Impossible to forget cleanup.

✅ **Builder pattern for filters**: Instead of filling C structures with pointers, you build rules with a fluent, typed API.

✅ **Automatic path conversion**: The crate transparently converts DOS paths to NT kernel paths. No more silently broken filters.

✅ **100% safe API**: The `unsafe` code (necessary to talk to Windows APIs) is isolated and encapsulated inside the crate. Users never need to write `unsafe`.

✅ **Network monitoring**: Subscribe to real-time network events via a Rust callback.

### **Example: Blocking an Application**

```rust
use windows_wfp::{WfpEngine, WfpProvider, WfpSublayer, WfpFilter};

fn main() -> Result<(), Box<dyn std::error::Error>> {
    // Open a WFP session (handle automatically closed via RAII)
    let engine = WfpEngine::open()?;

    // Register a provider (identifies who creates the rules)
    let provider = WfpProvider::new("My App", "Custom filtering rules");
    engine.register_provider(&provider)?;

    // Create a sublayer (rule group with priority)
    let sublayer = WfpSublayer::new("My Filters", 1000);
    engine.register_sublayer(&sublayer)?;

    // Block notepad.exe on TCP
    // The DOS path is automatically converted to NT kernel path
    let filter = WfpFilter::block()
        .name("Block Notepad")
        .application("C:\\Windows\\System32\\notepad.exe")
        .protocol(Protocol::Tcp)
        .build()?;

    engine.add_filter(&filter)?;

    println!("Filter active! notepad.exe is blocked on TCP.");
    Ok(())
}
```

The `?` at the end of each call is Rust's **error propagation operator**: if the call fails, the error is cleanly returned to the caller. No `try/catch`, no exceptions — everything goes through the type system.

### **Example: Real-Time Network Monitoring**

```rust
use windows_wfp::WfpMonitor;

fn main() -> Result<(), Box<dyn std::error::Error>> {
    let monitor = WfpMonitor::start(|event| {
        println!("Network event: {:?}", event);
    })?;

    // Monitoring runs in the background...
    std::thread::sleep(std::time::Duration::from_secs(60));

    monitor.stop()?;
    Ok(())
}
```

## 🔧 **Filtering Capabilities**

The crate supports a wide range of filtering conditions that can be combined:

| Condition | Description | Example |
|-----------|-------------|---------|
| **Application** | Filter by executable path | `notepad.exe`, `chrome.exe` |
| **Protocol** | TCP, UDP, ICMP, ICMPv6 | `Protocol::Tcp` |
| **Remote port** | Target server port | Port 443 (HTTPS) |
| **Local port** | Local application port | Port 8080 |
| **IP address** | IPv4/IPv6 with CIDR | `192.168.1.0/24` |
| **Windows service** | Filter by service name | `svchost`, `dns` |
| **AppContainer** | SID for UWP/Microsoft Store apps | Edge, packaged apps |

These conditions can be combined to create very precise rules. For example: block `chrome.exe` only on port 80 (HTTP) while allowing port 443 (HTTPS).

## 💡 **What I Learned Building This Crate**

### **Windows FFI in Rust is a World of Its Own**

Microsoft's `windows` crate has greatly simplified access to Windows APIs from Rust. But WFP remains a complex kernel API with nested structures, C unions, and sometimes under-documented behaviors. Each WFP filter involves building `FWPM_FILTER0`, `FWPM_FILTER_CONDITION0`, and similar structures with manual memory allocation and careful lifetime management.

### **Path Conversion is Critical**

The `FilterAPI.dll!FwpmGetAppIdFromFileName` function handles the DOS → NT kernel conversion, but it has its subtleties. Network paths (UNC), symbolic links, and environment variables are all edge cases to handle. It's the kind of invisible detail that makes the difference between a working filter and a ghost filter.

### **Network Monitoring Reveals a Lot**

While developing the monitoring feature, I was surprised by the volume and diversity of network traffic on a standard Windows workstation. Dozens of services communicate constantly — a reminder that the network attack surface is much broader than one might imagine.

## 🔮 **What's Next?**

**windows-wfp** is the first building block of a **larger project** I'm currently working on. A project related to network security on Windows, still in Rust. I won't say more for now... but stay tuned, it promises to be interesting. 😏

In the meantime, the crate is available and functional. Here's the roadmap for windows-wfp itself:

### **Upcoming Evolutions**
- 🔄 **Callout support**: Real-time packet interception and modification
- 📊 **Filtering statistics**: Blocked/allowed packet counters per filter
- 🧪 **Integration tests**: Automated test suite with real filter creation/deletion
- 📖 **Enhanced documentation**: Advanced usage guides and concrete use cases

## 🔗 **Resources and Links**

### **The Crate**
- 📦 **[crates.io](https://crates.io/crates/windows-wfp)**: Crate page
- 📖 **[docs.rs](https://docs.rs/windows-wfp)**: API documentation
- 🔗 **[GitHub](https://github.com/lostyzen/windows-wfp)**: Source code

### **Technical References**
- [Windows Filtering Platform - Microsoft Learn](https://learn.microsoft.com/en-us/windows/win32/fwp/windows-filtering-platform-start-page)
- [WFP Architecture Overview](https://learn.microsoft.com/en-us/windows/win32/fwp/windows-filtering-platform-architecture-overview)
- [Microsoft's `windows` crate](https://crates.io/crates/windows)

### **LinkedIn Post**
- 📣 **[LinkedIn Announcement](https://www.linkedin.com/posts/pierre-bazydlo_github-lostyzenwindows-wfp-safe-rust-activity-7437792839582765056-ZTPN)**: The post accompanying this release

## 🎉 **Conclusion**

Building **windows-wfp** allowed me to dive into the internals of Windows networking at the kernel level — a fascinating but demanding domain. The wrapper transforms a complex and trap-laden C API into something safe, ergonomic, and idiomatic in Rust.

If you work in network security on Windows, or if you're simply curious about how to properly wrap a kernel API in Rust, feel free to explore the code, open issues, or propose contributions.

**Questions? Feedback?** I welcome all feedback, whether about the API, documentation, or use cases you'd like to see supported.

📧 **Email**: [lostyzen@gmail.com](mailto:lostyzen@gmail.com)
🔐 **PGP Key**: [Download from keys.openpgp.org](https://keys.openpgp.org/search?q=lostyzen%40gmail.com)
📌 **Fingerprint**: `0205 9854 864D EE62 C2BB 455F D825 3521 EDD1 630B`
