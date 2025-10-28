![IMG_7767](https://github.com/user-attachments/assets/c702a844-305c-4593-be1a-9a6f82828eb2)



---

# Fixing bluetooth Audio (AAC) on Asahi Linux (aarch64).

**Target distro:** Fedora Linux Asahi Remix 42 (Forty Two [Adams])  
**Applies to:** Any Bluetooth headphones/earbuds using A2DP (AAC/SBC), e.g., AirPods Max, AirPods, Sony WH-1000XM series, Bose QC, etc.  
**Tested on:** MacBook Pro (Apple Silicon, M1)  
**Kernel:** 6.16.8-400.asahi.fc42.aarch64+16k

After applying the steps below, my AirPods Max now play flawlessly with AAC on Asahi Linux.

---

## Problem Summary
On Apple Silicon Macs running Fedora Asahi Remix, Bluetooth audio (especially AAC over A2DP) may stutter, pop, or drop packets when you’re on 2.4 GHz Wi-Fi. The system sometimes falls back to SBC or keeps AAC with severe glitching.

This issue doesn’t appear on macOS because Apple’s closed-source drivers handle RF coexistence automatically. On Linux, the open drivers are still evolving.

---

## Why This Happens
- Wi-Fi and Bluetooth share the same 2.4 GHz RF domain.
- Heavy Wi-Fi traffic can starve the BT scheduler.
- AAC requires stable bandwidth, and packet loss causes stuttering.

---

## What Works (Quick View)
1. Use 5 GHz Wi-Fi (best fix, no RF conflict)
2. Use a USB Wi-Fi adapter (stable and simple solution)
3. Optimize single-radio setups (mitigation only)

---

## Solution A — Use 5 GHz Wi-Fi (Best)
- Switch to a 5 GHz SSID (router or phone hotspot if supported).
- This separates Wi-Fi and Bluetooth into different frequency bands.
- AAC audio becomes stable with zero stutter.

---

## Solution B — Use an External USB Wi-Fi Adapter
Using a USB Wi-Fi adapter (e.g., Alfa AWUS036NHA – Atheros AR9271) eliminates coexistence issues.

```bash
# list devices
nmcli device

# disconnect internal wifi
nmcli device disconnect wlp1s0f0

# connect using USB wifi
nmcli dev wifi connect "YourSSID" password "YourPassword" ifname wlu1

# optional: turn off internal wifi radio
nmcli radio wifi off

# make the connection auto
nmcli connection modify "YourSSID" connection.autoconnect yes
```

**Result:** Bluetooth AAC plays smoothly without drops.

---

## Solution C — Optimize If You Can’t Use 5 GHz or USB

```bash
# List Bluetooth cards
pactl list cards | sed -n '/bluez_card\./,/^$/p'

# Force AAC
pactl set-card-profile bluez_card.<MAC_UNDERSCORED> a2dp-sink

# Verify active profile
pactl list cards | grep -i "Active Profile"
```

**Other optimizations:**

- **Lower AAC bitpool:**
  ```bash
  pactl set-sink-proplist bluez_output.<MAC_UNDERSCORED>.a2dp-sink bluetooth.a2dp.aac_bitpool=32
  ```

- **Reduce background Wi-Fi load** (disable updates, torrents, etc.)

- **Use a cleaner channel** (1 or 11)

- **Optional: lower Wi-Fi TX power** if close to the AP

---

## Quick Diagnostics

```bash
pactl list cards
nmcli device
nmcli dev wifi
```

**Symptoms:**

- Audio stutter or pops
- Profile flapping between AAC and SBC
- Playback works fine when Wi-Fi is disabled

---

## FAQ

**Q: Can I fix this fully without 5 GHz or USB?**  
A: Not entirely. Optimizations help, but coexistence remains.

**Q: Is this a PipeWire bug?**  
A: No. It’s primarily RF coexistence on Apple Silicon.

**Q: Will future updates fix it?**  
A: Likely yes. The Asahi stack is evolving quickly.

**Q: Does USB tethering help?**  
A: Yes, it removes Wi-Fi RF load entirely.

---

## TL;DR

- Use 5 GHz Wi-Fi whenever possible.
- Or use a USB Wi-Fi adapter (e.g., Alfa AWUS036NHA).
- Or tweak AAC and Wi-Fi settings to reduce stutter.
- Verified on AirPods Max with perfect AAC playback.

---

## Disclaimer

This guide targets Fedora Asahi Remix 42 on Apple Silicon. Use commands at your own risk. Some tweaks are situational; revert if performance degrades.
No affiliation with Apple, Fedora, or Alfa.

Contributions are welcome. If you find better tweaks or improvements, open an Issue or Pull Request.

---

