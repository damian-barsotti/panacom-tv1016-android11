# Guide: Upgrade firmware and configure the remote control on the Panacom TV-1016

Tutorial to replace the firmware of the **Panacom TV-1016** (SoC **Amlogic S905W2**, 2 GB RAM, 16 GB eMMC) with **Android 11**, and get the **original IR remote control** working with all of its keys, including the mouse-mode button.

> **Warning**
> This procedure wipes the device and modifies system partitions. There is a risk of bricking the device if something goes wrong. Do it at your own risk. Always keep the original boot image and the full firmware so you can recover with the USB Burning Tool.

---

## Table of contents

1. [Requirements](#1-requirements)
2. [Flash Android 11 (X98Q firmware)](#2-flash-android-11-x98q-firmware)
3. [First boot and WiFi](#3-first-boot-and-wifi)
   - [Optional: run the commands over SSH from your PC](#optional-run-the-commands-over-ssh-from-your-pc)
4. [Decode the IR remote control codes](#4-decode-the-ir-remote-control-codes)
5. [Install Magisk (root)](#5-install-magisk-root)
6. [Load the remote control persistently](#6-load-the-remote-control-persistently)
7. [Software mouse mode (optional)](#7-software-mouse-mode-optional)
8. [Final notes](#8-final-notes)

---

## 1. Requirements

- A **Windows PC** (for the USB Burning Tool).
- **Amlogic USB Burning Tool** v3.x — download: <https://androidmtk.com/download-amlogic-usb-burning-tool>
- **X98Q Android 11 firmware** (same hardware: S905W2, 2 GB / 16 GB) — download: <https://androidpctv.com/firmware-x98q-amlogic/>
- A **USB-A male to male cable** (USB-A to USB-A). It's not the usual charging cable; you can find it at computer/electronics stores.
- A **paperclip or thin pin** to press the reset button inside the AV port.
- On the device: the **Termux** app (to use the terminal with root).

> The X98Q firmware works because it shares the exact same SoC and memory configuration as the Panacom TV-1016. Other S905W2 firmwares (X98 Mini, etc.) may have incompatible WiFi drivers.

### Verify the chip (optional)

To confirm your unit actually has the **Amlogic S905W2**, you can open the TV box:

1. Turn the device over. You'll see **two glued-on rubber feet** on the base; under each one there's **a screw**. Peel off the rubber feet and remove the two screws.
2. Unscrew and separate the board from the case.
3. The SoC is the largest chip and has a **heatsink** on top. **Carefully peel it off** to read the chip's marking, which should say **Amlogic S905W2**.

> **When reassembling:** if the **thermal paste** between the chip and the heatsink has dried out or come off, **clean off the old residue** (from both the chip and the heatsink) and **apply fresh thermal paste** before putting the heatsink back. This keeps the SoC properly cooled.

---

## 2. Flash Android 11 (X98Q firmware)

### 2.1. Prepare the tool

1. Install the **Amlogic USB Burning Tool** (<https://androidmtk.com/download-amlogic-usb-burning-tool>) on Windows. During installation, also accept installing the **Amlogic USB drivers**.
2. Open the tool and load the firmware: **File → Import Image** and select the X98Q `.img` file.

### 2.2. Configure the options

- Under **Erase Flash**, choose **Normal Erase**.
- Check **Reset After Success**.
- Click **Start**. The tool will wait for you to connect the device.

### 2.3. Enter burn mode

The TV-1016 has the reset button **inside the AV port** (the round video connector).

1. With the device **powered off and disconnected**, insert the paperclip into the AV port and **hold down** the internal button.
2. Without releasing it, connect the **USB-A to USB-A cable** between the device and the PC.
3. Release the button after 3–5 seconds.
4. The tool detects the device and begins flashing.

> If it isn't detected, try the device's other USB port.

### 2.4. Finish

- Flashing takes 5 to 15 minutes. **Do not disconnect power during the process.**
- When it finishes it shows **"Burn successful"** and the device reboots on its own.
- The first boot may take several minutes. This is normal.

---

## 3. First boot and WiFi

1. Connect HDMI to the TV (or to the amplifier/receiver).
2. Complete the initial Android setup.
3. Connect to WiFi.

> **WiFi note:** if WiFi won't connect or is slow, it's usually the **antenna cable** poorly positioned inside the case. When closing the device, make sure the antenna cable isn't crushed or sharply bent. To measure the real speed, use a reliable tool (for example `speedtest-cli` from Termux); the in-browser tests on these boxes underestimate the speed.

### Update to newer versions (FOTA)

This firmware includes a **preinstalled FOTA update app** that lets you update to newer firmware versions directly from the device (over the air, without re-flashing with the USB Burning Tool). It's a good idea to check for updates from that app after the first boot.

> **Important:** if you're going to apply the root/remote-control setup in the following steps, do the FOTA updates first and only then install Magisk. A later FOTA update can overwrite the boot partition and disable Magisk (see [Final notes](#8-final-notes)).

### Install the apps

Apps that aren't in the store are installed as **APKs** (downloading them from the device's browser or transferring them by USB), for example Stremio and Tidal. Netflix is installed from the included store.

---

## Optional: run the commands over SSH from your PC

The commands in the following steps are run in **Termux**. Typing them on the TV box is awkward, so it's convenient to start an **SSH** server in Termux and connect from your PC (on the same WiFi network). That way you can comfortably copy and paste the commands from your PC's terminal.

### Steps in Termux (on the device)

1. Install the OpenSSH server:

   ```sh
   pkg install openssh
   ```

2. Set a password for the Termux user (without this, password login won't work):

   ```sh
   passwd
   ```

   Enter and confirm a password.

3. Find out the Termux **username** (you'll need it to connect):

   ```sh
   whoami
   ```

   It returns something like `u0_a51`.

4. Find out the device's **IP address** on the network:

   ```sh
   ifconfig wlan0
   ```

   Look for the address on the `inet` line (for example `192.168.0.42`). You can also see it under **Settings → WiFi → connected network**.

5. Start the SSH server:

   ```sh
   sshd
   ```

   > Termux listens on **port 8022** (not the standard 22). You have to run `sshd` every time you reboot, unless you automate it separately.

### Connect from your PC

With the username (step 3) and the IP (step 4), from a terminal on your PC (Linux/macOS, or Windows with OpenSSH):

```sh
ssh u0_a51@192.168.0.42 -p 8022
```

Replace `u0_a51` with your username and `192.168.0.42` with your device's IP. Enter the password you set in step 2. Once inside, you can run `su` and continue with the rest of the guide from your PC.

---

## 4. Decode the IR remote control codes

The Panacom remote uses a **custom code** that the X98Q firmware doesn't recognize out of the box. You need to read the remote's real codes and build a table.

### 4.1. Read the raw codes

1. Open **Termux** on the device.
2. Point the remote at the device and run the command, pressing each key right before:

   ```sh
   dmesg | grep -i "meson-ir"
   ```

   For each key, a line like this appears:

   ```
   meson-ir fe084040.ir: invalid custom:0xec137f80
   ```

   (It says "invalid" because the code isn't configured yet; that's exactly what we want at this stage.)

### 4.2. Interpret the code (little-endian)

The value `0xAABBCCDD` is read by **reversing the bytes**:

- **custom_code** = the **last 4 digits** of the value (the same for all keys).
- **scancode** = the **second byte** (3rd and 4th digits from the left), different for each key.

Example with the OK key = `0xec137f80`:

- custom_code = `0x7f80`
- scancode = `0x13`

### 4.3. Panacom remote table

These are the values read for this remote (yours may differ if you have a different unit — repeat step 4.1 for each button):

| Button | Raw value      | scancode | keycode (Linux) |
|--------|----------------|----------|------------------|
| Power  | 0x7e81**7f80** | 0x81     | 116 (POWER)      |
| Menu   | 0x7c83**7f80** | 0x83     | 125 (MENU)       |
| Home   | 0x8c73**7f80** | 0x73     | 102 (HOME)       |
| OK     | 0xec13**7f80** | 0x13     | 28 (ENTER/OK)    |
| Up     | 0xc738**7f80** | 0x38     | 103 (UP)         |
| Down   | 0xbf40**7f80** | 0x40     | 108 (DOWN)       |
| Left   | 0xc837**7f80** | 0x37     | 105 (LEFT)       |
| Right  | 0xc639**7f80** | 0x39     | 106 (RIGHT)      |
| Back   | 0xd827**7f80** | 0x27     | 158 (BACK)       |
| Vol−   | 0x7689**7f80** | 0x89     | 114 (VOL DOWN)   |
| Vol+   | 0x7887**7f80** | 0x87     | 115 (VOL UP)     |
| Mouse  | 0xb748**7f80** | 0x48     | 133 (INFO)       |

**Remote custom_code: `0x7f80`**

> The Linux keycodes are chosen so that the device's keylayout (`.kl`) file translates them into valid Android keys. The Mouse button is mapped to **133 (INFO)** because that keycode is mapped in the `.kl` and serves as the trigger for the software mouse app (see step 7).

---

## 5. Install Magisk (root)

Magisk is used to load the remote table at every boot in a "systemless" way (without modifying the real system partition).

> These steps assume the firmware provides root access (`su`). **Termux** is used for the commands.

### 5.1. Install the Magisk app

1. Download the official APK from **github.com/topjohnwu/Magisk/releases**.
2. Install it on the device (allowing "install apps from unknown sources").

### 5.2. Dump the boot image

In Termux:

```sh
su
SLOT=$(getprop ro.boot.slot_suffix)
dd if=/dev/block/by-name/boot$SLOT of=/sdcard/boot.img
```

> **Save a copy of `/sdcard/boot.img` on your PC.** It's your backup to revert Magisk with a `dd` if needed.

### 5.3. Patch the boot image from the command line

(On this hardware, patching from the app may give a "Process error"; the CLI method uses the same binaries shipped with the app and is reliable.)

```sh
su
APK=$(ls -d /data/app/*/com.topjohnwu.magisk*/ | head -1)
LIB=$(ls -d ${APK}lib/* | head -1)   # architecture folder (arm or arm64)

mkdir -p /data/local/tmp/mp && cd /data/local/tmp/mp
cp /sdcard/boot.img .

cp "$LIB/libmagiskboot.so"   ./magiskboot
cp "$LIB/libmagiskinit.so"   ./magiskinit
cp "$LIB/libmagiskpolicy.so" ./magiskpolicy
cp "$LIB/libbusybox.so"      ./busybox
cp "$LIB/libinit-ld.so"      ./init-ld   2>/dev/null
cp "$LIB/libmagisk.so"       ./magisk32  2>/dev/null
cp "$LIB/libmagisk.so"       ./magisk    2>/dev/null
chmod 755 ./*

./busybox unzip -o "${APK}base.apk" "assets/*" -d .
cp assets/boot_patch.sh assets/util_functions.sh assets/stub.apk .
chmod 755 boot_patch.sh

export KEEPVERITY=true KEEPFORCEENCRYPT=true RECOVERYMODE=false
sh boot_patch.sh boot.img
```

When it finishes, it generates **`new-boot.img`** in `/data/local/tmp/mp`.

### 5.4. Write the patched boot

```sh
su
SLOT=$(getprop ro.boot.slot_suffix)
dd if=/data/local/tmp/mp/new-boot.img of=/dev/block/by-name/boot$SLOT
sync
reboot
```

### 5.5. Complete the installation (Direct Install)

After rebooting, the Magisk daemon runs (`su` works), but the init integration still needs to be completed so the boot scripts run:

1. Open the **Magisk** app.
2. Tap **Install → Direct Install (Recommended)**.
   - **Do not** choose "Install to inactive slot".
3. Wait for "All done!" and **reboot**.

Verify that Magisk is active:

```sh
su
magisk -c        # should print the version, e.g. 30.7:MAGISK:R
```

---

## 6. Load the remote control persistently

The system loads the IR tables with the `/vendor/bin/remotecfg` binary. We'll create a Magisk module with our table and a script that loads it at every boot.

### 6.1. Create the module and the table file

```sh
su
MOD=/data/adb/modules/panacom_remote
mkdir -p $MOD/system/vendor/etc

cat > $MOD/module.prop <<'EOF'
id=panacom_remote
name=Panacom IR Remote
version=1.0
versionCode=1
author=user
description=Loads the Panacom remote (custom_code 0x7f80)
EOF

cat > $MOD/system/vendor/etc/remote.tab3 <<'EOF'
custom_name = amlogic-remote-3
custom_code = 0x7f80
release_delay = 80
fn_key_scancode = 0x00
cursor_left_scancode = 0x37
cursor_right_scancode = 0x39
cursor_up_scancode = 0x38
cursor_down_scancode = 0x40
cursor_ok_scancode = 0x13
key_begin
0x81 116
0x83 125
0x73 102
0x13 28
0x38 103
0x40 108
0x37 105
0x39 106
0x27 158
0x89 114
0x87 115
0x48 133
key_end
EOF

chmod 644 $MOD/module.prop $MOD/system/vendor/etc/remote.tab3
```

### 6.2. Create the boot-load script

```sh
su
cat > /data/adb/service.d/panacom_remote.sh <<'EOF'
#!/system/bin/sh
until [ "$(getprop sys.boot_completed)" = "1" ]; do sleep 2; done
sleep 5
/vendor/bin/remotecfg -t /data/adb/modules/panacom_remote/system/vendor/etc/remote.tab3
EOF

chmod 755 /data/adb/service.d/panacom_remote.sh
```

### 6.3. Test

Load the table on the fly (or reboot, which loads it automatically):

```sh
su
/vendor/bin/remotecfg -t /data/adb/modules/panacom_remote/system/vendor/etc/remote.tab3
cat /sys/class/remote/amremote/map_tables
```

You should see `0x7f80` in the list. Test all the remote's keys. Then reboot once to confirm the remote responds on its own, without running any command.

---

## 7. Software mouse mode (optional)

This firmware's kernel doesn't have the remote's hardware mouse mode, but you can get the same behavior (a pointer moved with the arrow keys) using an accessibility app. The **Mouse** button is already mapped to the **INFO keycode (165 on Android)** in step 6, ready to use as the trigger.

1. Download **MATVT** (Mouse for Android TV Toggle) from <https://github.com/virresh/matvt/releases>.
   - Use a **recent pre-release**; older stable versions have a bug that prevents clicking on some devices.
2. Install it and enable it under **Settings → Accessibility**, granting all permissions (including **"Display over other apps"**).
3. In the MATVT settings, use **"Automatically detect boss key code"** and press the **Mouse** button on the remote. It should detect **Key 165** (INFO).
4. Test it: the **Mouse** button toggles the cursor on/off, the arrow keys move it, and **OK** clicks.

> Keep a USB mouse on hand during setup in case the accessibility service gets misconfigured.

---

## 8. Final notes

- **Backup:** keep the original `boot.img` (step 5.2) in a safe place. It lets you revert Magisk with `dd` without resorting to the USB Burning Tool.
- **OTA updates:** an OTA can overwrite the boot partition and lose Magisk (and with it, the automatic loading of the remote). The module and the script in `/data` survive; you only need to redo Magisk's **Direct Install** (step 5.5) to reactivate everything.
- **Cleanup:** you can delete the temporary files when you're done:

  ```sh
  su
  rm -rf /data/local/tmp/mp
  rm -f /sdcard/boot.img
  ```

- **Customize keys:** to change a button's mapping, edit the file
  `/data/adb/modules/panacom_remote/system/vendor/etc/remote.tab3`
  and reboot (or reload with `remotecfg`).

---

*Values like the `custom_code` (`0x7f80`) and the scancodes are specific to this remote control. If your unit has a different remote, repeat step 4 to get your own.*
