---
title: "Complete Rust Development Environment Setup on Windows"
date: 2025-10-17T10:00:00+02:00
lastmod: 2025-10-17T10:00:00+02:00
draft: false
weight: 4
tags: ["Rust", "Windows", "Installation", "Development", "Scoop", "Toolchain", "MinGW", "Cargo", "No Admin", "User Install"]
categories: ["Tools", "Rust"]
author: "lostyzen"
showToc: true
TocOpen: false
hidemeta: false
comments: true
description: "Complete guide to install and configure a Rust development environment on Windows WITHOUT administrator rights: Scoop + GNU toolchain approach, with troubleshooting and practical examples."
canonicalURL: "https://lostyzen.github.io/posts/installation-environnement-rust-windows/"
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
    image: "/images/rust-windows-og.png" # Image spécifique à l'article
    alt: "Rust Windows Installation" # Texte alternatif
    caption: "Complete Rust environment setup on Windows"
    relative: false
    hidden: false
editPost:
    URL: "https://github.com/lostyzen/lostyzen.github.io/tree/main/content"
    Text: "Suggérer des changements"
    appendFilePath: true
# SEO Keywords (pour le contenu)
keywords: ["rust", "installation", "windows", "scoop", "mingw", "toolchain", "cargo", "development", "environment", "configuration", "no admin", "user rights", "user install", "no admin rights"]
# Schema.org structured data
schema:
  type: "BlogPosting"
  datePublished: "2025-10-17T10:00:00+02:00"
  dateModified: "2025-10-17T10:00:00+02:00"
  author: "lostyzen"
  publisher: "DevOps Blog"
# Open Graph / Facebook
images: ["/images/rust-windows-og.png"] # Image pour les réseaux sociaux
# Twitter Card (temporairement désactivé - réactiver quand comptes créés)
# twitter:
#   card: "summary_large_image"
#   site: "@votre_twitter"
#   creator: "@votre_twitter"
---

## 🎯 **Introduction**

Rust is a modern systems programming language that combines performance, memory safety, and productivity. In this article, we'll see how to install and configure a complete Rust development environment on Windows **WITHOUT administrator rights** using Scoop as a package manager.

**🎉 Great news:** This approach allows you to install Rust and all its dependencies without ever needing admin rights! Perfect for restrictive corporate environments or personal machines.

**Why this method?** Because traditional Rust installation often requires admin rights for MSVC dependencies. We'll elegantly bypass this with the GNU toolchain.

## 📋 **Prerequisites**

Before starting, make sure you have:

- **Windows 10/11** (recent versions recommended)
- **Scoop installed** (if not done yet, check [our Scoop article](https://lostyzen.github.io/posts/scoop-gestionnaire-windows/))
- **A terminal** (PowerShell, Windows Terminal, or Git Bash)
- **❌ NO administrator rights required!** Everything installs in your user directory

## 🔑 **Why this method works without admin rights?**

**Scoop's magic:**
- **User installation**: Scoop installs everything in `~/scoop/` (your personal directory)
- **No system modifications**: No changes in `C:\Program Files\` or Windows registry
- **Local PATH**: Executables are added to your user PATH only

**Advantages for restrictive environments:**
- ✅ Perfect for companies with strict security policies
- ✅ Ideal for shared computers or virtual machines
- ✅ Reversible installation: `scoop uninstall` removes everything cleanly
- ✅ No conflict with other system installations

**Comparison with traditional installation:**
| Method | Admin rights | System install | Reversible |
|--------|-------------|----------------|------------|
| **Scoop + GNU** | ❌ Not required | ❌ No | ✅ Yes |
| **Standard install** | ✅ Required | ✅ Yes | ❌ No |

Let's start the installation!

## 🚀 **Installing Rust via Scoop**

### **Step 1: Installing rustup-gnu**

```bash
scoop install rustup-gnu
```

**What does this command do?**
- Installs `rustup` configured to use the **GNU toolchain by default**
- Automatically configures the system PATH
- **Avoids having to change the toolchain** like with regular `rustup`
- Includes the GCC toolchain ready to use

**⚠️ Important:** Restart your terminal/VS Code after installation so PATH changes are taken into account.

### **Step 2: Installing MinGW-w64**

The GNU toolchain requires MinGW-w64 for compilation:

```bash
scoop install mingw
```

**What does this command do?**
- Installs MinGW-w64 (GNU for Windows)
- Provides necessary compilation tools (`gcc`, `g++`, `dlltool`, etc.)
- Automatically configures the PATH

**⚠️ Important:** Terminal restart required after MinGW installation.

### **Step 3: Verifying the installation**

```bash
# Check Rust
rustc --version

# Check Cargo (package manager)
cargo --version
```

**Expected result:**
```
rustc 1.90.0 (1159e78c4 2025-09-14)
cargo 1.90.0 (840b83a10 2025-07-30)
```

## 🔧 **Solving Common Issues**

### **Issue 1: Missing MSVC Dependencies**

If you get this error:
```
note: you may need to install Visual Studio build tools with the "C++ build tools" workload
```

**Solution:** This error should no longer appear with `rustup-gnu`, but if it occurs, explicitly use the GNU toolchain.

### **Issue 2: Missing dlltool**

If you get this error despite MinGW installation:
```
Error calling dlltool 'dlltool.exe': program not found
```

**Possible solutions:**
1. **Restart the terminal** (PATH not updated)
2. **Reinstall MinGW:**
```bash
scoop uninstall mingw
scoop install mingw
```
3. **Check the installation:**
```bash
dlltool --version
```

## 🏗️ **Setting up a Web Project with Axum**

For a modern web project, here's a basic configuration:

### **Project Initialization**

```bash
cargo init my-web-project
cd my-web-project
```

### **Cargo.toml Configuration**

```toml
[package]
name = "my-web-project"
version = "0.1.0"
edition = "2021"

[dependencies]
axum = "0.7"
tokio = { version = "1.0", features = ["full"] }
tower = "0.4"
serde = { version = "1.0", features = ["derive"] }
serde_json = "1.0"
```

### **Basic Server Example**

```rust
use axum::{
    routing::get,
    response::Json,
    Router,
};
use serde::Serialize;
use std::net::SocketAddr;

#[derive(Serialize)]
struct HealthCheck {
    status: String,
    message: String,
}

async fn health_check() -> Json<HealthCheck> {
    Json(HealthCheck {
        status: "ok".to_string(),
        message: "Server is running!".to_string(),
    })
}

#[tokio::main]
async fn main() {
    // Create the router
    let app = Router::new()
        .route("/health", get(health_check));

    // Start the server
    let addr = SocketAddr::from(([127, 0, 0, 1], 3000));
    println!("🚀 Server running on http://{}", addr);

    axum::Server::bind(&addr)
        .serve(app.into_make_service())
        .await
        .unwrap();
}
```

### **💡 Concrete Example**
To see these concepts in action, check out the demonstration repository that implements a complete Rust web server with Axum:

🔗 **[GitHub Repository - Rust Web Server Demo](https://github.com/lostyzen/rust-web-demo)**

## 📁 **Recommended Rust Project Structure**

Here's the structure of a modern Rust web project (based on our concrete example):

```
my-web-project/
├── Cargo.toml          # Rust project configuration
├── src/
│   ├── main.rs         # Application entry point
│   └── templates/      # HTML templates
│       └── home.html   # Home page template
└── README.md
```

**Key elements:**
- **`src/main.rs`**: Entry point with server logic
- **`src/templates/`**: HTML templates for views
- **`Cargo.toml`**: Dependencies and configuration

## 🛠️ **Essential Commands**

```bash
# Development
cargo build          # Debug compilation
cargo run            # Compile + run
cargo check          # Check without compilation

# Production
cargo build --release  # Optimized compilation

# Maintenance
cargo clean          # Clean artifacts
cargo update         # Update dependencies
cargo doc --open     # Generate documentation

# Testing
cargo test           # Run tests
cargo test --doc     # Documentation tests
```

## 🔍 **Advanced Troubleshooting**

### **Managing Multiple Toolchains**

```bash
# List installed toolchains
rustup toolchain list

# Temporarily change toolchain
rustup run stable-x86_64-pc-windows-gnu cargo build

# Install specific version
rustup install 1.89.0
```

### **Persistent PATH Issues**

If commands are still not found:

1. **Check PATH:**
```bash
echo $env:PATH
```

2. **Reinstall rustup:**
```bash
scoop uninstall rustup
scoop install rustup
```

3. **Complete Windows restart** if necessary

### **Conflicts with Other Rust Installations**

If you already installed Rust through other methods:

```bash
# Uninstall other versions
# Then cleanly reinstall with Scoop
scoop install rustup
```

## 🎯 **Best Practices**

### **1. Use Version Management**

```bash
# For projects, specify Rust version
# In Cargo.toml
[package]
rust-version = "1.70"
```

### **2. Optimal VS Code Configuration**

Create a `.vscode/settings.json` file:

```json
{
    "rust-analyzer.checkOnSave.command": "clippy",
    "rust-analyzer.server.path": "~/.cargo/bin/rust-analyzer",
    "editor.formatOnSave": true,
    "[rust]": {
        "editor.defaultFormatter": "rust-lang.rust-analyzer"
    }
}
```

### **3. Dependency Management**

- Use `cargo add` to add crates
- Regularly audit with `cargo audit`
- Keep dependencies updated with `cargo update`

## 🚀 **Next Steps**

Now that your environment is configured, you can:

1. **Explore the official documentation**: [doc.rust-lang.org](https://doc.rust-lang.org)
2. **Follow The Rust Book**: [doc.rust-lang.org/book](https://doc.rust-lang.org/book/)
3. **Join the community**: [users.rust-lang.org](https://users.rust-lang.org)
4. **Contribute to open source projects**

## 📚 **Recommended Resources**

- **📖 [The Rust Programming Language](https://doc.rust-lang.org/book/)** - The official book
- **🛠️ [Rustup Documentation](https://rust-lang.github.io/rustup/)** - Toolchain management
- **📦 [Crates.io](https://crates.io/)** - Rust package registry
- **💬 [Rust Discord](https://discord.gg/rust-lang)** - Active community

## 🎉 **Conclusion**

Congratulations! You now have a complete and functional Rust environment on Windows. The Scoop approach with GNU toolchain avoids MSVC dependency complications while maintaining excellent performance.

Rust is a demanding but extremely powerful language. Take time to explore its unique concepts like ownership, borrowing, and lifetimes.

**Your first project awaits!** 🚀

**Just starting with Rust?** Share your first impressions or difficulties encountered!