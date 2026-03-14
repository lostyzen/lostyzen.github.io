---
title: "windows-wfp : Un wrapper Rust safe pour le firewall kernel Windows"
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
description: "windows-wfp : crate Rust open-source offrant une API safe pour interagir avec la Windows Filtering Platform (WFP), le firewall kernel-level de Windows."
canonicalURL: "https://lostyzen.github.io/posts/windows-wfp-rust-crate/"
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
    alt: "windows-wfp : wrapper Rust pour Windows Filtering Platform"
    caption: "Un wrapper Rust safe pour le firewall kernel-level de Windows"
    relative: false
    hidden: false
editPost:
    URL: "https://github.com/lostyzen/lostyzen.github.io/tree/main/content"
    Text: "Suggérer des changements"
    appendFilePath: true
keywords: ["rust", "windows", "wfp", "windows filtering platform", "firewall", "kernel", "network security", "crate", "open source", "ffi", "safe wrapper", "envoy proxy", "packet filtering"]
schema:
  type: "BlogPosting"
  datePublished: "2026-03-14T10:00:00+02:00"
  dateModified: "2026-03-14T10:00:00+02:00"
  author: "lostyzen"
  publisher: "DevOps Blog"
images: ["/images/windows-wfp-rust-crate-og.png"]
---

🛡️ **windows-wfp : Un wrapper Rust safe pour le firewall kernel Windows**

Depuis quelques mois, je travaille sur un projet personnel en Rust lié à la sécurité réseau sous Windows. Un projet ambitieux dont je ne dévoilerai pas encore tous les détails — disons simplement qu'il nécessite d'**interagir directement avec le firewall kernel-level de Windows**. Et quand on plonge dans cet univers, on tombe inévitablement sur la **Windows Filtering Platform (WFP)**.

Le problème ? Manipuler WFP depuis Rust, c'est du sport. J'ai donc construit un wrapper pour mes propres besoins, et j'ai décidé de le partager en **open-source** sur crates.io : **[windows-wfp](https://crates.io/crates/windows-wfp)**.

## 🎯 **C'est quoi la Windows Filtering Platform ?**

La **WFP** (Windows Filtering Platform) est le framework de filtrage réseau au niveau kernel intégré à Windows depuis Vista/Server 2008. C'est le moteur qui fait tourner le **Pare-feu Windows** lui-même, ainsi que la plupart des solutions de sécurité réseau : antivirus, EDR, VPN...

Concrètement, WFP vous permet de :
- **Bloquer ou autoriser** du trafic réseau selon des règles fines
- **Filtrer par application** : autoriser Firefox mais bloquer une autre app
- **Filtrer par protocole, port, IP** : avec support CIDR pour les plages d'adresses
- **Monitorer en temps réel** les événements réseau du système
- **Intervenir au niveau kernel** : avant même que le trafic n'atteigne l'application

C'est une API extrêmement puissante, utilisée par les outils de sécurité professionnels. Mais elle a un gros défaut : elle est **conçue pour le C/C++**, et l'utiliser depuis Rust demande de jongler avec des abstractions bas-niveau peu ergonomiques.

## ⚠️ **Le problème : manipuler WFP sans wrapper**

Quand j'ai commencé à intégrer WFP dans mon projet Rust, je me suis vite retrouvé face à trois obstacles majeurs :

### **1. Du FFI brut partout**

Le **FFI** (Foreign Function Interface), c'est le mécanisme qui permet à Rust d'appeler les API C de Windows. Le problème, c'est qu'en passant par du FFI brut, on perd toutes les garanties de sécurité mémoire qui font la force de Rust. On manipule des pointeurs bruts, des structures C, et on se retrouve à écrire du code `unsafe` (c'est-à-dire du code où on dit au compilateur : "fais-moi confiance, je sais ce que je fais" — spoiler : on ne sait pas toujours ce qu'on fait 😅).

### **2. Des handles Windows non sécurisés**

Un **handle** sous Windows, c'est un identifiant opaque que le système vous donne quand vous ouvrez une ressource (une session WFP, un fichier, un processus...). Le souci : il faut penser à **fermer chaque handle manuellement**. Si on oublie → fuite de ressources. Si on l'utilise après l'avoir fermé → comportement indéfini. Le genre de bugs silencieux qui ne se manifestent qu'en production, 3 mois plus tard, à 2h du matin.

### **3. Le piège des chemins NT kernel**

C'est probablement le piège le plus vicieux. WFP opère au niveau kernel et utilise des **chemins NT kernel** (`\Device\HarddiskVolume3\Windows\System32\notepad.exe`), pas les chemins DOS habituels (`C:\Windows\System32\notepad.exe`).

Le résultat ? Vous ajoutez un filtre pour bloquer `notepad.exe` en utilisant son chemin classique. L'API WFP ne renvoie **aucune erreur** — tout semble fonctionner. Mais le filtre ne matche jamais le moindre paquet. Vos règles existent, mais elles sont muettes. 🙃

J'ai perdu un temps considérable à debugger ce comportement avant de comprendre le problème. C'est d'ailleurs ce genre de frustration qui m'a motivé à encapsuler toute cette complexité dans un wrapper propre.

## 🚀 **windows-wfp : l'approche Rust idiomatique**

Le crate **windows-wfp** encapsule toute cette complexité derrière une API safe, ergonomique et idiomatique en Rust.

### **Architecture du crate**

```
windows-wfp/
├── src/
│   ├── engine.rs       # Session WFP (RAII, auto-cleanup)
│   ├── provider.rs     # Enregistrement de providers
│   ├── sublayer.rs     # Gestion des sublayers (priorités)
│   ├── filter.rs       # Builder pattern pour les filtres
│   ├── condition.rs    # Conditions de filtrage
│   ├── path.rs         # Conversion DOS → NT kernel path
│   ├── monitor.rs      # Monitoring réseau temps réel
│   └── lib.rs          # Point d'entrée public
```

### **Les principes de conception**

✅ **RAII pour les handles** : Chaque session WFP, chaque provider, chaque filtre est wrappé dans un type Rust qui ferme automatiquement le handle quand l'objet sort du scope. Impossible d'oublier de nettoyer.

✅ **Builder pattern pour les filtres** : Au lieu de remplir des structures C avec des pointeurs, on construit les règles avec une API fluide et typée.

✅ **Conversion automatique des chemins** : Le crate convertit de façon transparente les chemins DOS en chemins NT kernel. Plus de filtres silencieusement cassés.

✅ **API 100% safe** : Le code `unsafe` (nécessaire pour parler aux API Windows) est isolé et encapsulé à l'intérieur du crate. L'utilisateur n'a jamais à écrire d'`unsafe`.

✅ **Monitoring réseau** : Souscription aux événements réseau en temps réel via un callback Rust.

### **Exemple : bloquer une application**

```rust
use windows_wfp::{WfpEngine, WfpProvider, WfpSublayer, WfpFilter};

fn main() -> Result<(), Box<dyn std::error::Error>> {
    // Ouvrir une session WFP (handle fermé automatiquement via RAII)
    let engine = WfpEngine::open()?;

    // Enregistrer un provider (identifie qui crée les règles)
    let provider = WfpProvider::new("Mon App", "Règles de filtrage custom");
    engine.register_provider(&provider)?;

    // Créer un sublayer (groupe de règles avec priorité)
    let sublayer = WfpSublayer::new("Mes Filtres", 1000);
    engine.register_sublayer(&sublayer)?;

    // Bloquer notepad.exe sur TCP
    // Le chemin DOS est automatiquement converti en chemin NT kernel
    let filter = WfpFilter::block()
        .name("Block Notepad")
        .application("C:\\Windows\\System32\\notepad.exe")
        .protocol(Protocol::Tcp)
        .build()?;

    engine.add_filter(&filter)?;

    println!("Filtre actif ! notepad.exe est bloqué sur TCP.");
    Ok(())
}
```

Le `?` à la fin de chaque appel, c'est l'**opérateur de propagation d'erreur** en Rust : si l'appel échoue, l'erreur est renvoyée proprement à l'appelant. Pas de `try/catch`, pas d'exceptions — tout passe par le système de types.

### **Exemple : monitorer le réseau en temps réel**

```rust
use windows_wfp::WfpMonitor;

fn main() -> Result<(), Box<dyn std::error::Error>> {
    let monitor = WfpMonitor::start(|event| {
        println!("Événement réseau : {:?}", event);
    })?;

    // Le monitoring tourne en arrière-plan...
    std::thread::sleep(std::time::Duration::from_secs(60));

    monitor.stop()?;
    Ok(())
}
```

## 🔧 **Capacités de filtrage**

Le crate supporte un large éventail de conditions de filtrage, combinables entre elles :

| Condition | Description | Exemple |
|-----------|-------------|---------|
| **Application** | Filtrer par chemin d'exécutable | `notepad.exe`, `chrome.exe` |
| **Protocole** | TCP, UDP, ICMP, ICMPv6 | `Protocol::Tcp` |
| **Port distant** | Port du serveur cible | Port 443 (HTTPS) |
| **Port local** | Port de l'application locale | Port 8080 |
| **Adresse IP** | IPv4/IPv6 avec CIDR | `192.168.1.0/24` |
| **Service Windows** | Filtrer par nom de service | `svchost`, `dns` |
| **AppContainer** | SID pour apps UWP/Microsoft Store | Edge, apps packagées |

Ces conditions peuvent être combinées pour créer des règles très précises. Par exemple : bloquer `chrome.exe` uniquement sur le port 80 (HTTP) tout en autorisant le port 443 (HTTPS).

## 💡 **Ce que j'ai appris en construisant ce crate**

### **Le FFI Windows en Rust, c'est un monde**

Le crate `windows` de Microsoft a énormément simplifié l'accès aux API Windows depuis Rust. Mais WFP reste une API kernel complexe avec des structures imbriquées, des unions C, et des comportements parfois sous-documentés. Chaque filtre WFP implique de construire des structures `FWPM_FILTER0`, `FWPM_FILTER_CONDITION0`, etc., avec des allocations mémoire manuelles et des durées de vie à gérer soigneusement.

### **La conversion de chemins est critique**

La fonction `FilterAPI.dll!FwpmGetAppIdFromFileName` fait la conversion DOS → NT kernel, mais elle a ses subtilités. Les chemins réseau (UNC), les liens symboliques, et les variables d'environnement sont autant de cas à gérer. C'est le genre de détail invisible qui fait la différence entre un filtre qui fonctionne et un filtre fantôme.

### **Le monitoring réseau révèle beaucoup de choses**

En développant la fonctionnalité de monitoring, j'ai été surpris de voir le volume et la diversité du trafic réseau sur un poste Windows classique. Des dizaines de services communiquent en permanence — c'est un rappel que la surface d'attaque réseau est bien plus large qu'on ne l'imagine.

## 🔮 **Et la suite ?**

**windows-wfp** est la première brique d'un **projet plus large** sur lequel je travaille actuellement. Un projet lié à la sécurité réseau sous Windows, toujours en Rust. Je n'en dis pas plus pour le moment... mais restez connectés, ça promet d'être intéressant. 😏

En attendant, le crate est disponible et fonctionnel. Voici la feuille de route pour windows-wfp lui-même :

### **Prochaines évolutions**
- 🔄 **Support des callouts** : Interception et modification de paquets en temps réel
- 📊 **Statistiques de filtrage** : Compteurs de paquets bloqués/autorisés par filtre
- 🧪 **Tests d'intégration** : Suite de tests automatisés avec création/suppression de filtres réels
- 📖 **Documentation enrichie** : Guides d'utilisation avancée et cas d'usage concrets

## 🔗 **Ressources et liens**

### **Le crate**
- 📦 **[crates.io](https://crates.io/crates/windows-wfp)** : Page du crate
- 📖 **[docs.rs](https://docs.rs/windows-wfp)** : Documentation API
- 🔗 **[GitHub](https://github.com/lostyzen/windows-wfp)** : Code source

### **Références techniques**
- [Windows Filtering Platform - Microsoft Learn](https://learn.microsoft.com/en-us/windows/win32/fwp/windows-filtering-platform-start-page)
- [WFP Architecture Overview](https://learn.microsoft.com/en-us/windows/win32/fwp/windows-filtering-platform-architecture-overview)
- [Le crate `windows` de Microsoft](https://crates.io/crates/windows)

### **Post LinkedIn**
- 📣 **[Annonce LinkedIn](https://www.linkedin.com/posts/pierre-bazydlo_github-lostyzenwindows-wfp-safe-rust-activity-7437792839582765056-ZTPN)** : Le post qui accompagne cette publication

## 🎉 **Conclusion**

Construire **windows-wfp** m'a permis de plonger dans les entrailles du réseau Windows au niveau kernel — un domaine fascinant mais exigeant. Le wrapper transforme une API C complexe et piégeuse en quelque chose de sûr, ergonomique et idiomatique en Rust.

Si vous travaillez sur de la sécurité réseau sous Windows, ou si vous êtes simplement curieux de voir comment on wrappe proprement une API kernel en Rust, n'hésitez pas à explorer le code, ouvrir des issues, ou proposer des contributions.

**Des questions ? Des retours ?** Je suis preneur de tous les feedbacks, que ce soit sur l'API, la documentation, ou les cas d'usage que vous aimeriez voir supportés.

📧 **Email** : [lostyzen@gmail.com](mailto:lostyzen@gmail.com)
🔐 **PGP Key** : [Download from keys.openpgp.org](https://keys.openpgp.org/search?q=lostyzen%40gmail.com)
📌 **Fingerprint** : `0205 9854 864D EE62 C2BB 455F D825 3521 EDD1 630B`
