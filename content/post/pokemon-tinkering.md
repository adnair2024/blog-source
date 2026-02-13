+++
date = '2026-02-12T18:04:06-06:00'
draft = false
title = 'Pokemon Tinkering'
type = 'post'
tags = ['coding','pokemon','tinkering']
categories = ['Fun', 'Tech', 'Hobby', 'Coding']
+++

# Shiny Patcher: A Deep Dive into ROM Manipulation and Pattern Matching!

![test](https://i.imgur.com/aTfXeSH.png)

Building a tool to modify legacy games requires more than just changing a few bytes; it is a delicate dance of reverse engineering, memory management, and robust pattern matching. The Shiny Patcher project is a Flask-based utility designed to give players control over their Pokémon experience—specifically targeting Gen 3 (GBA) and Gen 4 (NDS) titles—by allowing custom shiny rates and event flag toggling.

Here is the technical breakdown of how we solved the most complex hurdles during development.

---

## Project Overview
Shiny Patcher operates as a web-based bridge between raw ROM data and user-friendly toggles.
* **Backend:** Python with Flask for the web server.
* **ROM Handling:** ndspy for Nintendo DS file system manipulation and standard byte manipulation for GBA.
* **Frontend:** A modern, reactive interface built with Tailwind CSS, featuring a persistent Soft Reset (SR) counter stored in localStorage.

---

## Technical Challenges and Solutions

### 1. Eliminating False Positives in GBA Event Flags
One of the earliest hurdles was "ghost" events. In Pokémon FireRed, the scanner initially reported that the Aurora Ticket and Birth Island events were active on vanilla (unmodified) ROMs.

**The Fix:** We realized the scanner was simply checking if a byte was non-zero. Since vanilla ROMs often contain junk data or different metadata at those offsets, we moved to strict validation. The patcher now specifically looks for `0x01`—the exact byte our tool writes—to confirm a flag is active.

### 2. Solving the "Regex Parenthesis" Trap (NDS)
When searching for assembly patterns in NDS arm9.bin files, we encountered a frustrating `re.error: missing )`.

**The Cause:** In ARM assembly, the byte `0x28` represents a `CMP` instruction. However, in Regex, `0x28` is the ASCII code for `(`, which starts a capture group.

**The Solution:** We implemented escaped byte patterns and enabled `re.DOTALL` to ensure the binary data was handled as a single string of bytes.

### 3. Performance Optimization for HGSS
Nintendo DS ROMs (specifically HeartGold and SoulSilver) are significantly larger and more complex than GBA files. Initial versions of the patcher would hang because ndspy was decompressing and recompressing the arm9.bin file multiple times in a single request.

**The Refactor:**
* **Single-Pass Processing:** We modified the logic to load the ROM into memory once, perform all scans and patches, and then save the final state.
* **Compression Management:** By explicitly setting `rom.setArm9Compression(True)`, we prevented the output files from bloating to massive, unplayable sizes.

---

## Technical Specifications

### GBA FireRed (BPRE v1.0)
The GBA implementation relies on "Spec-First" detection. Rather than searching the whole ROM, we check the most reliable offsets first to ensure speed and accuracy.

| Feature | Specification |
| :--- | :--- |
| **Shiny Offset** | 0x39D5A (Primary), 0x39D5E (Secondary) |
| **Assembly Pattern** | XX 28 01 D2 (CMP R0, #XX; BHI) |
| **Event Flags** | 0x13A (Aurora), 0x8D5 (Birth Island) |

### Nintendo DS (HGSS IPKE/IPGE)
DS patching is more dynamic, requiring a targeted search within the ARM9 binary.

* **Search Range:** 0x60000 to 0x85000 within arm9.bin.
* **Pattern Matching:** 00 0C XX 28 01 D2.
* **Logic:** The code looks for the LSR (Logical Shift Right) instruction, which shifts the internal ID to prepare for the shiny comparison.

---

## Current Project Status
Shiny Patcher is now stable and highly accurate. It prioritizes hardcoded offsets for known versions while maintaining a robust fallback search for minor revisions. The UI remains lightweight, providing a scrolling marquee of supported titles and a persistent counter for hunters who prefer the long way—even if the rates are in their favor.

**What's Next?** Now that the core patching engine is solid, we can focus on expanding the Event Toggle library to include Gen 3 legendary encounters like Mew on Faraway Island.
