# Booting an OS on a HP ProLiant DL360 G6

## Introduction

I bought several rack servers to play with and eventually set up a small HPC.
While my **Dell PowerEdge R710** was relatively easy to get running, the **HP ProLiant DL360 G6** caused *a lot* of trouble.

It took me **over a week of daily attempts** just to boot an operating system.
Information online exists, but **no single source had everything needed**, so this guide is meant to close that gap.

### Important Note About USB Booting

If you (like me) tried to boot via:

* USB stick
* SD card

…you probably already wasted a lot of time.

I could not find *any* confirmed working method to boot this server from USB or SD.
If you *did* manage it somehow, please share it.

---

## The Real Problem: iLO Advanced License

If your HP server **did not ship with an iLO Advanced license**, you *must* buy one (≈ **11 € & 48h**).

Without it:

* ❌ No virtual media
* ❌ No ISO boot over network

With that out of the way, let’s start.

---

## Accessing the iLO Web Interface

First, you need access to the **iLO web interface**.

### Java Requirement (iLO2)

To do this, clone and follow the steps in this repo:

👉 [https://github.com/niclan/Javafox.git](https://github.com/niclan/Javafox.git)

Once the **Javafox container** is running and you can access the browser:

* Confirm the security warnings
* Enter the iLO IP address

### Finding the iLO IP

* In your router, the device is usually named something containing **iLO**
* You can also configure iLO directly:

  * Connect a monitor
  * Press **F8 during boot**
  * (On my server the F8 hint was off-screen, but it still worked)
  * Set static ip
  * add new user

---

## iLO Login Issues (Important)

Normally, credentials are printed on a sticker:

* **User:** `Administrator`
* **Password:** printed on the chassis

⚠️ **In my case this did not work**, even after resetting BIOS settings.

### Fix

* Enter the iLO setup via **F8**
* Create a **new iLO user**
* Log in using the new credentials

After that, the iLO2 web interface worked.

---

## Firmware Versions (iLO & BIOS)

Once logged in, check:

* iLO version
* BIOS / ROM version

In my case, **everything was extremely outdated**.

### iLO Update (Optional but Recommended)

Updating iLO is not strictly required, but recommended.

If your iLO version is **below 2.0**, update **step by step**:

Example upgrade path that worked for me:

```
1.00 → 1.50 → 1.80 → 2.00 → 2.33
```

⚠️ Avoid skipping major versions.

You can upload `.rpm` files via:

```
iLO Web GUI → Administration → Firmware Update
```

---

## BIOS / ROM Update (Strongly Recommended)

The BIOS has a **backup ROM**, so updating it is safer than usual.

### HP SPP ISO

Officially, BIOS updates require an HP service contract.
However, this ISO worked for me:

**HP SPP 2017.04.0**

* Source: [https://archive.org/details/spp2017040](https://archive.org/details/spp2017040)
* Size: ~6.6 GB

### ISO Checksums

```text
Filename:
871795_001_spp-2017.04.0-SPP2017040.2017_0420.14.iso

MD5:
e9e85699bc1ebacfcb3d79def4124854

SHA1:
406ae4c7c64945a842b6c3124646a66c8f27135d

SHA256:
a434acaac2561169c1bd78f2c02177c43e7eb5ab0b35c049f54d47969f138d43
```

### Verify on Windows (PowerShell)

```powershell
CertUtil -hashfile 871795_001_spp-2017.04.0-SPP2017040.2017_0420.14.iso MD5
CertUtil -hashfile 871795_001_spp-2017.04.0-SPP2017040.2017_0420.14.iso SHA1
CertUtil -hashfile 871795_001_spp-2017.04.0-SPP2017040.2017_0420.14.iso SHA256
```

---

## iLO Advanced License (Required)

To mount ISOs via virtual media, you **must** activate iLO Advanced.

I bought mine here (cheap and worked fine):

👉 [https://www.etsy.com/de/listing/1434315324/hpe-hp-ilo-erweiterte-lizenz-lebenslange](https://www.etsy.com/de/listing/1434315324/hpe-hp-ilo-erweiterte-lizenz-lebenslange)

* License arrived within **48 hours**
* Enter it under **Activate Key**

---

## Mounting the ISO via Virtual Media

After activating the license:

1. Go to **Virtual Media**
2. Accept the security warnings
3. A new window opens
4. Select **Local File**
5. Choose the SPP ISO
   (from WSL or wherever your Javafox container runs)
6. Click **Connect**

🔴 Red indicator → 🟢 Green indicator = success

---

## Booting the Server

1. Reboot the server
2. Press **F11** for one-time boot menu
3. Select:

   ```
   (1) CD
   ```
4. The server now boots from the mounted ISO

Run through the SPP update process.

---

## Installing an Operating System

After firmware updates:

* Disconnect the SPP ISO
* Mount your OS ISO via virtual media

### Tested ISOs

| OS             | Status                     |
| -------------- | -------------------------- |
| Ubuntu 24.04.3 | ❌ NIC issues               |
| Ubuntu 22.04.5 | ✅ Works                    |
| RHEL 9.6       | ❌ Hardware incompatibility |
| Rocky Linux 9  | 🔜 Planned                 |

⚠️ Booting is **very slow** compared to a Dell R710 — seems normal for this HP generation.

---

## Final Notes

I hope this saves you a *lot* of time.

I plan to:

* Add screenshots
* Test more Linux ISOs

…but right now university has priority, and this server already consumed enough hours 😄

Good luck 🍀