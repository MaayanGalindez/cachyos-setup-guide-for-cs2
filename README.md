# CS2 on Linux: CachyOS Setup Guide (AMD & NVIDIA)

Nov 10, 2026 · @Maayan

## What this guide sets up

**A desktop PC that runs CS2 on CachyOS Linux with the KDE Plasma desktop, set up for the lowest latency your hardware allows.** Follow Steps 1–9 in order. Each step makes one choice for you and says why; anything optional is at the end.

**Why CachyOS:** it is Arch Linux with its packages rebuilt for modern CPUs (CachyOS measures 5–20% over generic builds, [optimized repos](https://wiki.cachyos.org/features/optimized_repos/)), a performance-tuned kernel, and the newest graphics drivers within days of release. It became the most-reported distro on ProtonDB in March 2026 ([ComputingForGeeks](https://computingforgeeks.com/best-linux-desktop-distributions/)). Expect a small gain from the distro itself: about 4% over Bazzite in one test ([Tech Insider](https://tech-insider.org/bazzite-vs-cachyos-2026/)). The settings in Steps 6–9 matter more.

**This guide is not for you if:**

- **You play FACEIT, ESEA or Valorant.** Their anti-cheat runs only on Windows 10 and 11. Valve matchmaking, Premier and VAC work fine on Linux.
- **You never want to use a terminal.** CachyOS needs a weekly update command. Bazzite is the safer pick for that.
- **You have a laptop.** This guide assumes a desktop PC with one graphics card. Laptops with two GPUs need extra setup it doesn't cover.
- **You want Windows and Linux on the same drive.** Step 3 erases the whole drive. Windows on a *separate* drive is fine: unplug that drive for the install (Step 1), and pick it from the BIOS boot menu when you need Windows.

**GeForce GTX 900/1000 owners:** it works, but on NVIDIA's frozen legacy driver (Step 5).

**Time:** about 1–2 hours, most of it downloads. **Grey boxes** hold text to copy exactly: mostly terminal commands, sometimes a Steam launch line or file content (the step says which). Lines starting with `#` are comments.

## Pick your path

**Answer three questions. Everything else is the same for everyone.**

**1. Which graphics card do you have?**

| Your card | Examples | What changes for you |
| --- | --- | --- |
| AMD Radeon RX 400 or newer | RX 580, RX 6800, RX 9070 XT | Follow every **AMD** line in Steps 5, 8 and 9 |
| NVIDIA RTX 20 or newer, or GTX 16 | GTX 1660, RTX 3060, RTX 5090 | Follow every **NVIDIA** line in Steps 5, 8 and 9 |
| NVIDIA GTX 900 or GTX 1000 | GTX 970, GTX 1080 Ti | Follow the **NVIDIA** lines; Step 5 explains the legacy driver |
| NVIDIA GTX 700 or older | GTX 780 | Not recommended for CS2 on Linux |

**2. Which processor do you have?**

- **AMD Ryzen:** turn on CPPC in the BIOS (Step 2).
- **Intel Core:** nothing extra.

**3. Which desktop?**

- **KDE Plasma:** the main guide. If you're unsure, pick this.
- **Hyprland** (keyboard-driven tiling): choose it in Step 3, then use Appendix A in place of Step 6.

**Not sure what you have?** In any Linux terminal, including the live USB:

```bash
lscpu | grep "Model name"           # CPU
lspci -k | grep -EA3 "VGA|3D"       # GPU
```

## Step 1. Prepare in Windows

Do this while Windows still works. Installing CachyOS erases the drive.

1. **Back up your CS2 config** to a USB stick or cloud drive. Steam is usually in `C:\Program Files (x86)\Steam`. Copy the folder `Steam\userdata\<your-id>\730\local\cfg\` (`<your-id>` is a folder named with a number; there's usually only one), plus any `autoexec.cfg` in `Steam\steamapps\common\Counter-Strike Global Offensive\game\csgo\cfg\`. You'll restore it in Step 8.
2. **Back up anything else** you care about. Steam Cloud covers most game saves, not all.
3. **Save mouse and keyboard settings to the device.** Apps like Logitech G HUB and Razer Synapse don't run on Linux. Set your DPI, polling rate and buttons now and save them to the device's onboard memory, so they keep working without the app.
4. **Update the BIOS (recommended, not required).** Newer firmware fixes bugs and improves CPU support. Do it before Step 2, because an update resets BIOS settings. Skip it if you'd rather not: an interrupted update can leave the board unable to start.
   1. Find your motherboard model: press Win+R, type `msinfo32`, read "BaseBoard Product".
   2. Download the newest BIOS for that model from the board maker's support page.
   3. Unzip it onto a FAT32-formatted USB stick.
   4. Reboot into the BIOS (Del or F2 at power-on) and open its flash tool: ASUS EZ Flash, MSI M-Flash, Gigabyte Q-Flash, ASRock Instant Flash.
   5. Pick the file and let it finish. Don't switch off while it flashes.
5. **Download CachyOS:** from [cachyos.org/download](https://cachyos.org/download/), the **Desktop** edition ISO plus its `.sha256` file.
6. **Verify the download** so a broken file doesn't fail the install halfway. In File Explorer, open your Downloads folder, click the address bar, type `powershell` and press Enter. Then run:

   ```powershell
   Get-FileHash .\cachyos-desktop-linux-*.iso -Algorithm SHA256
   ```

   The long code it prints must match the code inside the `.sha256` file (open it with Notepad). Capital and small letters don't matter. If the codes differ, download again.
7. **Write the USB** (8 GB or larger; it will be erased): open [Rufus](https://rufus.ie/), pick the stick under **Device**, click **SELECT** and choose the ISO, then **START** ([CachyOS wiki](https://wiki.cachyos.org/installation/installation_prepare/)). If Rufus asks about an "ISOHybrid" image, keep **ISO Image mode**. If the PC later won't boot from the stick, write it again and choose **DD Image mode**.
8. **Plan for Ethernet.** The installer downloads packages live and fails if Wi-Fi drops.
9. **Unplug other drives** (the SATA data cable) so the wrong one can't be wiped. Reconnect them after Step 3. M.2 drives are harder to remove: instead, write down the exact size of the drive you're installing to, and check it in Step 3.

## Step 2. BIOS settings

Reboot and press Del or F2 to enter the BIOS. Menu names vary by board; look under Advanced, Boot, AMD CBS or Overclocking. Save and exit with F10 when done.

| Setting | Set to | Who | Why |
| --- | --- | --- | --- |
| CSM / Legacy boot | Disabled | Everyone | CachyOS installs in UEFI mode only |
| Secure Boot | Disabled | Everyone | The USB won't boot with it on. CS2 and VAC don't need it. ASUS boards: set OS Type to "Other OS" |
| Storage (SATA) mode | AHCI; on Intel boards also turn VMD / RST off | Everyone | In RAID, RST or VMD mode the installer may not see your drive |
| Memory profile (EXPO / XMP / DOCP) | Enabled | Everyone | Without it, RAM runs below its rated speed, and CS2 at high fps is sensitive to memory speed |
| Above 4G Decoding | Enabled | Everyone | Required for the next setting |
| Re-Size BAR | Enabled (if listed) | Everyone | Lets the CPU reach all of the GPU's memory at once |
| CPPC and CPPC Preferred Cores | Enabled or Auto | AMD Ryzen | Lets Linux use its modern AMD CPU driver, which boosts faster |
| Fast Boot | Disabled | Everyone | Fast Boot can skip USB devices, including the install stick |

**Can't find a setting?** Leave it. Only the first two rows are required for the install; the rest are improvements.

## Step 3. Install CachyOS

The installer takes about ten minutes ([ComputingForGeeks walkthrough](https://computingforgeeks.com/install-cachyos-step-by-step/)). Plug in Ethernet first. Nothing is erased until you confirm on the Summary screen, so you can go back and change any answer before that.

1. **Boot the USB:** at power-on, tap your board's boot-menu key (usually F8 on ASUS, F11 on MSI and ASRock, F12 on Gigabyte), pick the USB stick, then the default **CachyOS** entry. If the screen stays black (most often with older NVIDIA cards), restart and pick **Legacy Hardware (GPU nomodeset)** instead.
2. **Start the installer:** the live desktop opens CachyOS Hello; click **Launch installer**.
3. **Welcome, Location, Keyboard:** pick your language, timezone and keyboard layout. Type a few characters, including symbols, in the test box. A wrong layout is the usual reason a password fails at the first login.
4. **Bootloader: Limine** (already selected). It shows Btrfs snapshots in the boot menu, which lets you undo a bad update ([boot managers](https://wiki.cachyos.org/installation/boot_managers/)).
5. **Partitions:** under **Select storage device**, choose the drive you're installing to and check its size; the USB stick is in the same list. Choose **Erase disk** and keep the filesystem on **btrfs**, which is what makes snapshots possible. Leave **Encrypt system** unticked unless you want to type a passphrase at every boot.
6. **Desktop: Plasma** (already selected), or Hyprland if you chose Appendix A.
7. **Packages:** leave the defaults. Gaming packages come in Step 8.
8. **Users:** enter your name, a login name, a computer name and a password. Leave "Use the same password for the administrator account" ticked. You'll type this password for every `sudo` command later.
9. **Summary:** check that the drive listed is the right one, then click **Install** and **Install Now**. The installer detects your graphics card and installs its driver automatically.
10. **Finish:** tick **Restart now**, click **Done**, and pull out the USB stick when the screen goes black.

## Step 4. First boot

**1. Log in** with the account you created. Keep the session on the login screen set to **Plasma (Wayland)**, the default. CachyOS Hello opens; you can close it, since this guide installs everything from the terminal.

**2. Open a terminal:** press **Ctrl+Alt+T** (Hyprland: **Super+Return**, Super being the Windows key).

- Paste with **Ctrl+Shift+V**, copy with **Ctrl+Shift+C**. Plain Ctrl+V doesn't paste in a terminal.
- Run commands one line at a time, top to bottom, and wait for each to finish.
- `sudo` commands ask for your password. Nothing appears while you type it; that's normal. Press Enter.
- When a command asks `[Y/n]`, press Enter to say yes.
- The shell (fish) suggests commands as you type; press → to accept a suggestion.

**3. Update everything, then restart:**

```bash
sudo pacman -Syu
reboot
```

**4. Check the basics** after the restart:

```bash
echo $XDG_SESSION_TYPE                    # must print: wayland
grep '^\[cachyos' /etc/pacman.conf        # shows which CPU-optimized repos you got (v3, v4 or znver4)
```

**5. Check the safety-net kernel.** The kernel is the core of Linux, and the installer adds a second, older one (`linux-cachyos-lts`) next to the main one. If a future update breaks the main kernel, you boot this one instead (Keeping it running explains how). Confirm it's there:

```bash
pacman -Q linux-cachyos-lts
```

It should print the name and a version number. If it says "was not found", open **CachyOS Kernel Manager** from the app menu, tick `linux-cachyos-lts` and click **Execute**. The app also installs the extra pieces a kernel needs, such as the NVIDIA driver module.

**6. Confirm snapshots work.** Every update now saves a before/after snapshot you can roll back to. This should list a few entries, including "Fresh CachyOS Installation":

```bash
sudo snapper -c root list
```

**Old snapshots are deleted automatically**, so they don't fill your drive. Snapper (the snapshot tool) keeps the newest 50, deletes older ones in a once-a-day cleanup, and removes before/after pairs where nothing changed. The boot menu lists only the newest few (about 8–10, depending on space on the boot partition), dropping the oldest entry when a new one arrives ([CachyOS wiki](https://wiki.cachyos.org/configuration/btrfs_snapshots/)). Check that the daily cleanup is switched on:

```bash
systemctl is-enabled snapper-cleanup.timer
```

If it prints `disabled`, turn it on:

```bash
sudo systemctl enable --now snapper-cleanup.timer
```

**Optional: keep 10 instead of 50.** CachyOS recommends about 10, because older snapshots are rarely useful and reverting to a very old one can cause problems with newer software. This also frees disk space:

```bash
sudo snapper -c root set-config NUMBER_LIMIT=10 NUMBER_LIMIT_IMPORTANT=10
```

## Step 5. Check your graphics driver

**The installer already set up your driver. This step only confirms it.** First install the checking tool:

```bash
sudo pacman -S --needed vulkan-tools
```

**AMD Radeon**

Your driver is Mesa's RADV, which AMD itself has backed as its official Vulkan driver since discontinuing AMDVLK in September 2025 ([Phoronix](https://www.phoronix.com/news/AMDVLK-Discontinued)).

```bash
vulkaninfo --summary | grep -i devicename   # your card, with "RADV" in the name
pacman -Qs amdvlk                           # must print nothing
```

If the first line shows `llvmpipe` instead of your card, the driver isn't loading; run `sudo chwd -a` and reboot.

**NVIDIA GeForce**

The installer picked the driver for your card's generation:

| Your card | Driver | What it means |
| --- | --- | --- |
| RTX 50 series | Current driver, open kernel modules | The only option for these cards |
| RTX 20/30/40, GTX 16 | Current driver, open kernel modules | Best-supported path |
| GTX 900 / GTX 1000 | Legacy `nvidia-580xx` | NVIDIA's 590 driver dropped these cards; 580 gets security fixes only, until October 2028 ([Igor'sLAB](https://www.igorslab.de/en/arch-linux-is-phasing-out-pascal-gtx-10-cards-are-being-removed-from-the-default-path-with-nvidia-590/)) |

```bash
nvidia-smi                                  # lists your card and driver version
vulkaninfo --summary | grep -i devicename   # your GeForce, not llvmpipe
```

If `nvidia-smi` errors, run `sudo chwd -a` and reboot. One NVIDIA difference matters later: its driver lets native Wayland games tear (lowest latency) but not XWayland ones ([KDE bug 487403](https://bugs.kde.org/show_bug.cgi?id=487403)), so Step 9 runs CS2 in native Wayland mode on NVIDIA.

**Driver updates** come with `sudo pacman -Syu` for both brands; there is no separate driver installer.

## Step 6. Display settings

Open **System Settings → Display & Monitor**. Click a monitor in the picture at the top to choose which one you're editing, set it as below, then click **Apply**. (Hyprland users: use Appendix A instead.)

| Setting | Gaming monitor | Any other monitor | Why |
| --- | --- | --- | --- |
| Refresh rate | The highest listed | Its normal rate | Plasma runs each screen at its own rate, so a 60 Hz side screen doesn't slow the main one |
| Primary | Yes | No | Games open on the primary screen. If CS2 thinks a 60 Hz screen is primary, it only offers 60 Hz. With two screens it's a **Primary** option; with three or more, click **Change Screen Priorities** and put the gaming monitor first |
| Adaptive Sync | Automatic | Never | Automatic turns VRR (FreeSync/G-Sync) on only for fullscreen games, so the desktop is unaffected. Side screens get Never because VRR can make them flicker |
| Scale | 100% | Any | Avoids scaling the game's image |

**Then turn on tearing.** With the gaming monitor still selected, switch on **Screen tearing: Allow in fullscreen windows** and click **Apply**. Tearing shows each new frame immediately instead of waiting for the screen. It is the lowest-latency mode, which is what you want in CS2, and it only ever affects a fullscreen game, never the desktop.

**Something missing?**

- **The highest refresh rate isn't listed:** connect the monitor with DisplayPort, or the HDMI port and cable its manual names for high refresh.
- **There's no Adaptive Sync row:** turn on FreeSync / Adaptive-Sync / G-Sync in the monitor's own on-screen menu, then reopen this page.

**How VRR and tearing work together:** when CS2 runs below your refresh rate, VRR keeps it smooth. When it runs above (common in CS2), tearing takes over for the lowest input lag. You don't need to choose between them.

**One monitor?** Ignore the Primary row and the right-hand column.

## Step 7. System settings for gaming

**1. Confirm the CPU driver**

```bash
cat /sys/devices/system/cpu/cpufreq/policy0/scaling_driver
```

| CPU | Should print | If not |
| --- | --- | --- |
| AMD Ryzen 3000 or newer | `amd-pstate-epp` | You see `acpi-cpufreq`: CPPC is off. Go back to Step 2 and enable it |
| AMD Ryzen 1000 / 2000 | `acpi-cpufreq` | Nothing to do; these CPUs don't support the newer driver |
| Intel Core | `intel_pstate` | Nothing to do |

**2. Shader cache: nothing to set.** A shader cache stores the graphics programs your GPU has already compiled, so CS2 doesn't recompile them mid-match. Steam already manages it: when it launches a game with Shader Pre-Caching on (Step 8), it points the driver's cache to `steamapps/shadercache/730` and sets the limits itself ([CS2 launch environment](https://gist.github.com/apprehensions/5d5f95347f69644ba7a6c93dab2a0173), [variables Steam sets](https://github.com/ValveSoftware/Proton/issues/8926)):

- **NVIDIA:** Steam turns off the driver's 1 GB cache cleanup (`__GL_SHADER_DISK_CACHE_SKIP_CLEANUP=1`), so the cache is never wiped.
- **AMD:** Steam uses Mesa's single-file cache, which has no size limit to hit ([Mesa docs](https://docs.mesa3d.org/envvars)).

CachyOS's tip to raise the cache size applies only if you turn Steam's pre-caching off ([gaming guide](https://wiki.cachyos.org/configuration/gaming/)). This guide keeps it on, so leave this alone. One rare exception on NVIDIA: if CS2 recompiles shaders on every launch, that's a known Steam bug with very large caches ([steam-for-linux #13622](https://github.com/ValveSoftware/steam-for-linux/issues/13622)), not a setting you missed.

**3. Turn off the file indexer.** Plasma's indexer (Baloo) scans your files in the background, using disk and CPU you'd rather give CS2:

```bash
balooctl6 disable
```

**4****. Optional: a second drive for games.** Skip this if CS2 goes on your main drive. Use ext4, not NTFS; CachyOS warns games on NTFS behave unpredictably. Reconnect the drive if you unplugged it. **Formatting erases everything on it.**

1. **Find it:** run `lsblk -f` and look for the right size with no `/` or `/home` mount point. NVMe drives look like `nvme1n1`, SATA drives like `sdb`.
2. **Format it** (replace `nvme1n1` with yours; on SATA the partition is `sdb1` instead of `nvme1n1p1`):

   ```bash
   sudo pacman -S --needed parted
   sudo wipefs -a /dev/nvme1n1
   sudo parted -s /dev/nvme1n1 mklabel gpt mkpart games ext4 0% 100%
   sudo mkfs.ext4 -L games /dev/nvme1n1p1
   ```
3. **Mount it at every boot.** This creates `/mnt/games` and adds the drive to `/etc/fstab`. `nofail` lets the PC still boot if the drive is ever missing:

   ```bash
   sudo mkdir -p /mnt/games
   echo "UUID=$(sudo blkid -s UUID -o value /dev/nvme1n1p1) /mnt/games ext4 defaults,noatime,nofail 0 2" | sudo tee -a /etc/fstab
   sudo systemctl daemon-reload
   sudo mount -a
   sudo chown $USER: /mnt/games
   ```
4. **Check:** `lsblk -f` should now show `/mnt/games` next to the partition. You'll add it to Steam in Step 8.

If `mount -a` reports an error, open the file with `sudo nano /etc/fstab`, delete the last line, save (Ctrl+O, Enter), exit (Ctrl+X), and repeat sub-step 3.

## Step 8. Install Steam and CS2

**1. Install Steam with pacman.** Installed this way, Steam's package and libraries update with the rest of the system on every `sudo pacman -Syu`. Run the line for your card:

```bash
# AMD Radeon
sudo pacman -S --needed steam lib32-vulkan-radeon

# NVIDIA GTX 16 / RTX 20 or newer
sudo pacman -S --needed steam lib32-nvidia-utils

# NVIDIA GTX 900 / GTX 1000
sudo pacman -S --needed steam lib32-nvidia-580xx-utils
```

The `lib32-` package is your card's 32-bit driver, which Steam itself needs. If pacman asks you to pick a provider for anything else, press Enter. Don't use the Flatpak version of Steam; it updates outside pacman.

**2. First launch.** Open **Steam** from the app menu. It downloads its own runtime first (a few hundred MB; Steam keeps its client updated itself, which is normal). Then log in and approve with Steam Guard.

**3. Steam settings** (Steam → Settings):

| Where | Set to | Why |
| --- | --- | --- |
| Downloads → Shader Pre-Caching | Both switches **On** | Shaders get compiled before you play instead of mid-match, which is what causes stutter (see below) |
| Game Recording | **Off** | Background recording costs performance; CachyOS documents stutter caused by it ([gaming guide](https://wiki.cachyos.org/configuration/gaming/)) |
| Compatibility | Leave as it is | Only affects Windows games, not CS2 |

**Why shader pre-caching stays on.** It doesn't change your fps once shaders are compiled; it changes *when* they compile. With it on, Steam downloads CS2's shader data and compiles it in the background while Steam is idle, so that work is done before a match instead of during one. A CachyOS user who turned it off for CS2 got heavy stutter and fps drops, which went away when he turned it back on ([CachyOS forum](https://discuss.cachyos.org/t/cs2-triggers-1-5gb-2gb-updates-on-every-reboot-nvidia-shader-cache-persistence-issue-on-btrfs/32804)).

The costs are bandwidth and idle CPU, not in-game speed. CS2 may download 1.5–2 GB of shader data after updates or reboots, and Steam uses CPU while idle to process it, then drops back once a game is running ([steam-for-linux #9266](https://github.com/ValveSoftware/steam-for-linux/issues/9266)). To get the full benefit, leave Steam open for a few minutes after an update, and if "Processing Vulkan shaders" appears when you launch CS2, let it finish instead of skipping it.

**4. Games drive (only if you set one up in Step 7):** Settings → Storage → drive drop-down → **Add Drive** → `/mnt/games`. Then open its ⋮ menu → **Make Default**.

**5. Install CS2:** Library → search **Counter-Strike 2** → **Install**. When it's done, right-click CS2 → Properties → **Compatibility** and make sure the box is **unticked**. CS2 has a native Linux version; running it through Proton (Steam's Windows layer) makes things worse, and players report VAC refusing matchmaking under Proton ([Turbosmurfs](https://turbosmurfs.gg/article/cs2-on-linux)).

**6. Restore your config.** Start CS2 once and close it, so its folders exist. Steam Cloud usually brings back your keybinds and game settings by itself; copy files from your Step 1 backup only for what's missing (an `autoexec.cfg` never syncs):

| From Windows | To Linux |
| --- | --- |
| `Steam\userdata\<id>\730\local\cfg\` | `~/.local/share/Steam/userdata/<id>/730/local/cfg/` |
| `...\game\csgo\cfg\autoexec.cfg` | `<your Steam library>/steamapps/common/Counter-Strike Global Offensive/game/csgo/cfg/` |

- **Finding the folders:** open the **Dolphin** file manager. `~` means your Home folder, and `.local` is hidden, so press **Ctrl+H** to show hidden folders. For the second row there's a shortcut: in Steam, right-click CS2 → Manage → **Browse local files**, then open `game/csgo/cfg`.
- **Your Steam library** is `~/.local/share/Steam` on the main drive, or `/mnt/games/SteamLibrary` on a games drive.
- **If CS2 opens with a black screen or the wrong resolution** after copying, delete `cs2_video.txt` from the first folder. It holds the Windows display settings, and CS2 creates a fresh one.

## Step 9. Set up CS2

**1. Wayland or XWayland: how CS2 reaches your screen**

**What they are.** Your desktop runs on **Wayland**, the modern display system; from Plasma 6.8 it is the only session KDE ships ([KDE blog](https://blogs.kde.org/2025/11/26/going-all-in-on-a-wayland-future/)). Apps built for Wayland talk to it directly. Older apps built for X11 (the previous system) run through **XWayland**, a translation layer that Plasma starts by itself inside your Wayland desktop. You never log into an X11 session or install anything for it. Steam itself is an X11 app, so XWayland is always running next to your games.

**Where it's decided.** CS2 can run either way, and the choice is one launch option. Valve's start script (`cs2.sh`) picks XWayland unless you set `SDL_VIDEO_DRIVER` yourself; Valve tried defaulting to Wayland in July 2025 and reverted after customer issues, but left the option open ([script quoted here](https://davidjusto.com/articles/m2p-latency/)).

- **XWayland:** no option needed (Valve's default).
- **Native Wayland:** `SDL_VIDEO_DRIVER=wayland`. Mind the underscore: the older name `SDL_VIDEODRIVER` that many guides show is overruled by the script, so CS2 silently stays on XWayland.

**How it affects you**

|  | XWayland (default) | Native Wayland |
| --- | --- | --- |
| Input latency, AMD | Same as native: on par with Wayland and X11 in KWin developers' measurements ([cited by David Justo](https://davidjusto.com/articles/m2p-latency/)) | Same |
| Input latency, NVIDIA | About double: \~14 ms vs \~7 ms in a measured RTX 4090 test with tearing allowed ([David Justo](https://davidjusto.com/articles/m2p-latency/)) | \~7 ms, the same as X11 and Windows |
| Tearing, AMD | Yes, with the KDE window rule in part 4 | Yes, with the Step 6 switch alone |
| Tearing, NVIDIA | No: NVIDIA's driver only tears for native Wayland games ([KDE bug 487403](https://bugs.kde.org/show_bug.cgi?id=487403)) | Yes |
| CS2 display modes | Fullscreen, Fullscreen Windowed, Windowed | Plain Fullscreen may be missing from the list. Not a loss: on Wayland, Plasma treats any window that fills the screen the same way, tearing included |
| Steam overlay (Shift+Tab) | Works | Doesn't work; Steam's overlay doesn't support native Wayland yet |
| VRR | Works | Works |
| 4:3 stretched | Black bars unless you use Gamescope | Players report it stretches on its own |
| Known issues | A stuck-mouse bug in CS2's bundled SDL under XWayland, reported mostly on Ubuntu/GNOME and still open in November 2026 ([fix-cs2-wayland](https://github.com/git-on-with-it/fix-cs2-wayland-linux)) | Less tested, since it isn't Valve's default; some players report frame-time spikes |

**What this guide uses**

- **AMD → XWayland** plus the tearing window rule (part 4): the same latency as native, Valve's tested default, and the Steam overlay keeps working. Switch to native Wayland if your mouse sticks or you play 4:3 stretched.
- **NVIDIA → native Wayland:** it's the only way to get tearing, and it halves input latency on NVIDIA. Switch back to XWayland only if CS2 crashes or misbehaves.

**NVIDIA: tearing needs four things, with no window rule:**

1. **Screen tearing: Allow in fullscreen windows** is on (Step 6). Plasma never tears without it.
2. **CS2 runs on native Wayland** (`SDL_VIDEO_DRIVER=wayland` in part 2). NVIDIA's driver doesn't tear for XWayland games.
3. **CS2's V-Sync is off** (part 3). With V-Sync on, CS2 asks to wait for the screen, so it never tears.
4. **CS2 fills the whole screen** with nothing drawn on top of it. A notification or volume pop-up pauses tearing while it shows.

**If you never see tear lines on NVIDIA** (check in part 6): go through the four points again, then update with `sudo pacman -Syu` and restart, because tearing on NVIDIA depends on the driver version. If it still doesn't tear, leave it. CS2 is still on the low-latency native Wayland path, and VRR keeps it smooth.

**How to tell which one CS2 is using:** while CS2 is running, press Alt+Tab to switch to a terminal and run:

```bash
tr '\0' '\n' < /proc/$(pgrep -xo cs2)/environ | grep SDL_VIDEO_DRIVER
```

It reads the setting CS2 was started with, and prints `SDL_VIDEO_DRIVER=x11` for XWayland or `SDL_VIDEO_DRIVER=wayland` for native Wayland.

**2. Launch options.** Right-click CS2 → Properties → **Launch Options**, and paste the line for your card:

```bash
# AMD Radeon (XWayland)
game-performance %command% -fullscreen

# NVIDIA GeForce (native Wayland)
SDL_VIDEO_DRIVER=wayland game-performance %command% -fullscreen
```

| Part | What it does |
| --- | --- |
| `SDL_VIDEO_DRIVER=wayland` | Runs CS2 as a native Wayland app (part 1) |
| `game-performance` | Switches the PC to its Performance power profile while CS2 runs, and back when you quit ([gaming guide](https://wiki.cachyos.org/configuration/gaming/)) |
| `%command%` | Steam's placeholder for the game. Settings before it change how the game starts; settings after it go to the game |
| `-fullscreen` | Starts CS2 fullscreen, avoiding a screen-flicker bug with several monitors |

Add nothing else. CS2 on Linux only uses Vulkan, so `-vulkan` does nothing, and old Windows options like `-high` or `-threads` don't help.

**3. In-game settings** (Settings → Video and Settings → Game):

- **Display Mode: Fullscreen.** If plain Fullscreen isn't in the list (this can happen in native Wayland mode), pick **Fullscreen Windowed**; on Wayland the two behave the same.
- **Refresh Rate:** the highest offered. If the option is greyed out, the game uses the rate you set in Step 6.
- **Advanced Video → Wait for Vertical Sync (V-Sync): Disabled.** V-Sync makes CS2 wait for the screen, which adds input lag and blocks tearing. VRR (Step 6) keeps things smooth without it.
- **Graphics quality:** the same settings you used on Windows.
- **Game → Telemetry → Show frame time and FPS: Always.** This is your fps counter for the checks below ([csdb.gg](https://csdb.gg/guides/how-to-show-fps-cs2/)).

**4. AMD only: allow tearing for CS2.** On AMD, CS2 runs through XWayland, which Plasma's tearing switch doesn't cover on its own ([r/linux\_gaming thread](https://reddit.sentinel-team.org/posts/1r8vwo3/snapshots/2026-02-20T03%3A23%3A09.523908Z)). A window rule tells Plasma to allow tearing for CS2 specifically:

1. With CS2 running, set its Display Mode to **Windowed** for a moment, so you can click on its window.
2. Open System Settings → Window Management → **Window Rules** → **Add New**, and type a description such as "CS2 tearing".
3. Click **Detect Window Properties**, then click anywhere on the CS2 window. In the list that opens, choose **Window class (application)**, so the rule applies only to CS2.
4. Click **Add Property**, search for "tearing", add **Allow tearing**, and set it to **Force** and **Yes**.
5. Click **Apply**, then set CS2 back to **Fullscreen**.

**5. Play two matches.** CS2 compiles shaders during the first runs, so some stutter is expected at first. It happens again briefly after driver updates.

**6. Check that everything works**

| Check | Good result | If not |
| --- | --- | --- |
| Which mode CS2 runs in (the command in part 1) | AMD: `x11`. NVIDIA: `wayland` | NVIDIA: check the launch line says `SDL_VIDEO_DRIVER` with the underscore |
| Refresh rate | Your monitor's maximum | Another monitor is primary: fix Primary in Step 6 |
| FPS counter during a match | Goes above your refresh rate (e.g. 300+ on a 240 Hz screen), so nothing is capping it | Stuck at the refresh rate: turn off Wait for Vertical Sync (part 3). If you're in Fullscreen Windowed and plain Fullscreen is offered, switch to it |
| Tearing | Turning fast on a bright map, you briefly see thin horizontal lines where the image is split. That's tearing working: the lowest-latency mode | No lines: check the Step 6 tearing switch and V-Sync off; AMD: the window rule (part 4); NVIDIA: the four-point list in part 1 |
| Mouse | Moves normally and stays in the game | Stuck near the center: a known XWayland bug, so use the NVIDIA line (native Wayland) on any card. NVIDIA mouse escaping or crashes: use the AMD line (XWayland) |
| Sound | Works | Add `SDL_AUDIO_DRIVER=pipewire` at the very start of the launch line |
| Third match onward | No regular stutter | See Keeping it running → Troubleshooting |

**7. Only if you play 4:3 stretched.** On Wayland a game can't change the monitor's resolution, so by default a 4:3 image gets black bars.

1. Use the **NVIDIA** launch line, whatever your card. In native Wayland mode, players report that 4:3 resolutions stretch to fill the screen. If the mouse then feels wrong, use the Gamescope option below.
2. In CS2's video settings, set Aspect Ratio to **4:3** and pick your 4:3 resolution.
3. **Still black bars?** Use Gamescope, Valve's small compositor that stretches the image itself. Install it and replace your launch line (set `-W/-H` to your monitor, `-w/-h` to your 4:3 resolution, `-r` to your refresh rate):

```bash
sudo pacman -S gamescope
```

```bash
game-performance gamescope -W 1920 -H 1080 -w 1440 -h 1080 -S stretch -f -r 144 --force-grab-cursor --immediate-flips -- %command%
```

## Optional: squeeze out more performance

**Only start this once Step 9 works.** Change one thing at a time, measure it, and keep it only if it wins. CachyOS itself notes that software tweaks rarely give double-digit fps gains ([gaming guide](https://wiki.cachyos.org/configuration/gaming/)).

**A. How to measure (needed for B–D)**

1. In CS2, turn on the console: Settings → Game → **Enable Developer Console (\~): Yes**.
2. Subscribe to the workshop map [CS2 FPS BENCHMARK DUST2](https://steamcommunity.com/sharedfiles/filedetails/?id=3240880604). It flies through the same scripted scene every time, which makes runs comparable.
3. Start it: Play → Workshop Maps → the benchmark map → Go. Don't touch anything while it runs.
4. When it ends, press **\~** to open the console and find the line that looks like `[VProf] FPS: Avg=343.9, P1=154.0`. **Avg** is your average fps. **P1** is your 1% low: the slowest 1% of frames, which is what you feel as stutter.
5. Do three runs for each setup and write the numbers down.

**How to decide:** keep a change only if P1 is better in all three runs. Differences of a few percent between runs are normal noise.

**B. Try the BORE kernel.** The default kernel (`linux-cachyos`) uses EEVDF, the standard Linux CPU scheduler with CachyOS tuning. `linux-cachyos-bore` uses BORE, a scheduler that favors responsiveness when the CPU is busy ([kernel guide](https://computingforgeeks.com/cachyos-kernel-manager-performance-tuning/)). In one CachyOS forum test, BORE cut missed frames under heavy CPU load from 5.3 to 0.6 per 1,000, while average fps barely moved ([forum thread](https://discuss.cachyos.org/t/kernels-trying-to-understand/24023)). It isn't a guaranteed win: the default kernel is built with extra compiler optimizations that the BORE build lacks. That's why you measure. CPUs with fewer cores are most likely to benefit.

1. Benchmark first on the default kernel. `uname -r` shows which kernel is running.
2. Install BORE alongside it: open **CachyOS Kernel Manager** from the app menu, tick `linux-cachyos-bore` and click **Execute**.
3. Restart. When the Limine boot menu appears, press an arrow key to stop the countdown, then pick `linux-cachyos-bore` under the **CachyOS** entry.
4. **NVIDIA only:** run `nvidia-smi`. If it lists your card, carry on. If it gives an error, this kernel has no NVIDIA driver module: restart, pick the normal kernel, and skip this test.
5. Benchmark again. Limine remembers the last entry you booted, so just keep booting the winner.

**C. Try SMT (Hyper-Threading) off.** One player reported about 100 fps more in CS2 on an 8-core CPU ([Turbosmurfs](https://turbosmurfs.gg/article/cs2-on-linux)); on 6 cores or fewer it is more likely to hurt.

1. Turn it off until the next restart: `echo off | sudo tee /sys/devices/system/cpu/smt/control`
2. Benchmark. If it loses, just restart and it's back on.
3. If it wins, make it permanent. These two commands add one line to the boot settings and rebuild the boot menu:

   ```bash
   echo 'KERNEL_CMDLINE[default]+=" nosmt"' | sudo tee -a /etc/default/limine
   sudo limine-mkinitcpio
   ```

   To undo it later, open the file with `sudo nano /etc/default/limine`, delete that last line, save (Ctrl+O, Enter), exit (Ctrl+X), and run `sudo limine-mkinitcpio` again.

**D. Cap the frame rate (only if fps swings a lot).** A steady frame rate can feel smoother than a higher one that jumps around. If your P1 is far below your average (less than half of it), try a cap near your P1 value: open the console and type, for example, `fps_max 300`. CS2 remembers it, and `fps_max 0` removes the cap. Otherwise leave it uncapped.

**Don't bother with:** the real-time (`rt`) kernel, which CachyOS says lowers gaming performance ([kernel page](https://wiki.cachyos.org/de/features/kernel)), or turning off CPU security protections.

## Keeping it running

**Once a week, update everything and restart:**

```bash
sudo pacman -Syu
reboot
```

Always update the whole system this way. Never install a single package with `pacman -Sy <name>`; partial updates are unsupported on Arch and can break things. If you'd like a tray reminder when updates are waiting, run `cachy-update --check --enable` once.

**If an update gives a black screen: boot the safety-net kernel**

1. Restart. When the Limine boot menu appears, press an arrow key to stop the countdown. Under the **CachyOS** entry, pick `linux-cachyos-lts` and press Enter.
2. Once on the desktop, run `sudo pacman -Syu` again; fixes usually arrive within a day or two.
3. On a later restart, pick your normal kernel (`linux-cachyos`) again.

**If an update breaks something else: roll back with a snapshot**

1. Restart and press an arrow key to stop the countdown. Under the CachyOS entry, open **Snapshots** and pick one from before the update. Each is labeled with its date and the update that created it.
2. The system starts from that snapshot and shows **"Restore this snapshot now"**. Accept it. If you miss it, open **Limine-snapper-restore** from the app menu, or run `sudo limine-snapper-restore`, and pick the same snapshot.
3. Restart and choose the normal **CachyOS** entry, not the snapshot ([CachyOS forum](https://discuss.cachyos.org/t/update-linux-kernel-7-2-gives-black-screen-feedback-snapshot-restore-works-but-the-recovery-ux-is-confusing/34579)).
4. Wait a day or two before updating again.

A snapshot can't undo a broken kernel; that's what the safety-net kernel is for.

**Troubleshooting**

| Problem | Fix |
| --- | --- |
| Stutter in the first matches | Shaders compiling. Play each map twice; it recurs briefly after driver updates |
| VAC error, can't queue | CS2 → Properties → Compatibility must be unticked (Step 8) |
| CS2 opens on the wrong monitor, or only offers 60 Hz | Make the gaming monitor Primary (Step 6) |
| Fps stuck exactly at refresh rate | Turn off Wait for Vertical Sync in CS2; NVIDIA: confirm CS2 runs in native Wayland (Step 9 checks) |
| Freezes when focused, runs when alt-tabbed | Use the AMD launch line, or the Gamescope line from Step 9 |
| No sound | Pick the output in Plasma's audio settings, run `systemctl --user restart pipewire wireplumber`, then try `SDL_AUDIO_DRIVER=pipewire` at the start of the launch line |
| Monitor blanks briefly in fullscreen | Set that monitor's Adaptive Sync to Never (Step 6) |
| `nvidia-smi` reports a version mismatch | You updated without restarting. Restart |
| Fps lower than on Windows | Re-check Step 5 (real GPU, not llvmpipe), Step 7 (CPU driver), and that the launch line includes `game-performance` |
| Mouse stuck near the center of the screen | A known bug in CS2 under XWayland. Use the NVIDIA launch line (native Wayland) on any card (Step 9) |
| CS2 doesn't start after you changed the launch options | Check the line for typos. Clear the Launch Options box to confirm CS2 starts without it, then paste the line again |
| "Processing Vulkan shaders" takes long at launch | Normal once after each game or driver update; let it finish. If it repeats at every launch on NVIDIA, it's the known Steam bug described in Step 7 |

Most CS2 fixes above come from [Turbosmurfs' CS2 Linux guide](https://turbosmurfs.gg/article/cs2-on-linux). For anything else, the [CachyOS forum](https://discuss.cachyos.org/) is active and helpful.

## Switching GPU brand later

**No reinstall needed.** CachyOS's hardware tool (chwd) swaps the driver in a few commands ([GPU migration guide](https://wiki.cachyos.org/features/chwd/gpu_migration/)). Run `sudo pacman -Syu` first.

| Step | AMD → NVIDIA | NVIDIA → AMD |
| --- | --- | --- |
| 1. Remove the old driver | `sudo chwd -r amd` | `sudo chwd --list-installed`, then `sudo chwd -r <profile-name>` |
| 2. Swap the card | Shut down, install the new card, boot | Same |
| 3. Install the new driver | `sudo chwd -a` | `sudo chwd -a` |
| 4. Rebuild boot entries | `sudo limine-mkinitcpio`, then restart | Same |
| 5. Verify | `nvidia-smi` lists the card | `vulkaninfo --summary \| grep -i devicename` shows RADV |
| 6. Steam's 32-bit driver | `sudo pacman -S --needed lib32-nvidia-utils` | `sudo pacman -S --needed lib32-vulkan-radeon` |
| 7. CS2 launch line (Step 9) | Switch to the NVIDIA line | Switch to the AMD line and add the AMD window rule |

**Upgrading from a GTX 900/1000 to a newer GeForce:** the legacy driver isn't replaced on its own. Run `sudo chwd -r nvidia-dkms-580xx`, then `sudo chwd -a` ([CachyOS forum](https://discuss.cachyos.org/t/graphics-card-upgrade-nvidia/27420)), and install `lib32-nvidia-utils`.

Afterwards, redo the checks at the end of Step 9.

## Appendix A: Hyprland instead of KDE Plasma

**Choose Hyprland only if you want a keyboard-driven tiling desktop set up in text files.** In fullscreen CS2 it gives no meaningful fps advantage over Plasma. Pick it in Step 3, follow every step as written, but use this appendix instead of Step 6, and skip the AMD window rule in Step 9 (the rule below covers it).

**Keys for day one** (Super = Windows key; from CachyOS's [Hyprland guide](https://wiki.cachyos.org/configuration/desktop_environments/hyprland/)):

| Action | Keys |
| --- | --- |
| Terminal | Super+Return |
| App launcher (Steam, settings, anything) | Super+Space |
| Close window | Super+Q |
| Fullscreen window | Super+F |
| Desktop settings | Super+Z |
| Lock / power menu | Super+L / Super+Alt+C |

**Where the settings live:** `~/.config/hypr/config/`. CachyOS's setup uses Hyprland's Lua format (Hyprland 0.55+), and changes apply as soon as you save.

**1. Monitors.** Run `hyprctl monitors` to see each monitor's name (like `DP-1`) and supported modes. Put the names in `variables.lua`, then set each monitor in `monitors.lua`:

```lua
-- Gaming monitor at full refresh; side monitor to its right with VRR off
hl.monitor({ output = "DP-1",     mode = "2560x1440@240", position = "0x0",    scale = 1 })
hl.monitor({ output = "HDMI-A-1", mode = "1920x1080@60",  position = "2560x0", scale = 1, vrr = 0 })
```

The first monitor change may need a restart. **VRR:** CachyOS's setup already enables it for fullscreen games only (`vrr = 3`, [Hyprland wiki](https://wiki.hypr.land/configuring/core/monitors/modes/)), the same behavior as Plasma's Automatic. Leave it; set `vrr = 0` only on a monitor that flickers.

**2. Tearing for CS2.** Add both lines to `windowrules.lua` ([Hyprland tearing wiki](https://wiki.hypr.land/Configuring/Advanced-and-Cool/Tearing/)):

```lua
hl.config({ general = { allow_tearing = true } })
hl.window_rule({ match = { class = "cs2" }, immediate = true })
hl.window_rule({ match = { class = "SDL Application" }, immediate = true })
```

The last line covers native Wayland mode (the NVIDIA launch line), where CS2's window has been reported under the name "SDL Application". To see the name on your system, run `hyprctl clients` while CS2 is open and look at its `class` line. Tearing only applies while CS2 is fullscreen and nothing else (bar, notification) is drawn on that monitor. Then continue with Step 7.

## Appendix B: Optional tools

**None of these are needed for CS2.** Install one only when you have the need in the third column.

| Tool | What it does | When you'd want it | Install |
| --- | --- | --- | --- |
| MangoHud | Overlay and logging for fps and frame times | A live fps and frame-time graph while you play: add `mangohud` right after `game-performance` in the launch line | `sudo pacman -S mangohud` |
| Gamescope | Valve's mini compositor: scaling, stretching, frame limits | 4:3 stretched with black bars (Step 9) | `sudo pacman -S gamescope` |
| LACT | GPU power limit, fan curve, clocks and undervolting; AMD and NVIDIA | A quieter or cooler GPU, or manual tuning | `sudo pacman -S lact` |
| nvtop | Live GPU usage in a terminal | Checking whether the GPU is fully used | `sudo pacman -S nvtop` |
| Piper | DPI, polling rate and button settings for Logitech, SteelSeries and other supported mice | Changing mouse settings without the Windows app | `sudo pacman -S piper` |
| Btrfs Assistant | Point-and-click snapshot browser and restore | If you prefer a GUI over the rollback steps | `sudo pacman -S btrfs-assistant` |
| sched-ext schedulers | Swap the CPU scheduler without restarting | Experimenting only; results vary by CPU, and in one test scx\_lavd made frame pacing 16 times worse ([forum](https://discuss.cachyos.org/t/kernels-trying-to-understand/24023)) | `sudo pacman -S scx-scheds scx-tools` |
| CachyOS gaming bundle | Lutris, Heroic, Proton-CachyOS and more | Playing Windows or non-Steam games | `sudo pacman -S cachyos-gaming-meta cachyos-gaming-applications` |

**LACT setup.** Start its background service once with `sudo systemctl enable --now lactd`, then open LACT from the app menu.

- **AMD:** power-limit and voltage controls need a kernel option first. Run these two commands, then restart:

  ```bash
  echo 'KERNEL_CMDLINE[default]+=" amdgpu.ppfeaturemask=0xffffffff"' | sudo tee -a /etc/default/limine
  sudo limine-mkinitcpio
  ```
- **NVIDIA:** works with NVIDIA's own driver. Undervolting is indirect: combine a clock offset with a locked maximum clock ([LACT 0.7.2 notes](https://linuxcompatible.org/story/lact-072-released)).

Change one value at a time and test in CS2. An unstable setting shows up as crashes or driver resets.

## Sources

- [CS2 on Linux in 2026: Native, VAC Works, FACEIT Doesn't](https://turbosmurfs.gg/article/cs2-on-linux) (Turbosmurfs, 29 Oct 2026)
- [Gaming with CachyOS Guide](https://wiki.cachyos.org/configuration/gaming/) (CachyOS wiki)
- [Optimized Repositories](https://wiki.cachyos.org/features/optimized_repos/) (CachyOS wiki)
- [CachyOS Kernel](https://wiki.cachyos.org/de/features/kernel) (CachyOS wiki)
- [Offered Boot Managers](https://wiki.cachyos.org/installation/boot_managers/) (CachyOS wiki)
- [Switching Between NVIDIA and AMD GPUs](https://wiki.cachyos.org/features/chwd/gpu_migration/) (CachyOS wiki)
- [Hyprland Post Install Setup & Keybinds](https://wiki.cachyos.org/configuration/desktop_environments/hyprland/) (CachyOS wiki)
- [Kernels - Trying to understand](https://discuss.cachyos.org/t/kernels-trying-to-understand/24023) (CachyOS forum, BORE test, Aug 2026)
- [Snapshot restore walkthrough](https://discuss.cachyos.org/t/update-linux-kernel-7-2-gives-black-screen-feedback-snapshot-restore-works-but-the-recovery-ux-is-confusing/34579) (CachyOS forum)
- [Graphics Card Upgrade Nvidia](https://discuss.cachyos.org/t/graphics-card-upgrade-nvidia/27420) (CachyOS forum)
- [Mesa environment variables](https://docs.mesa3d.org/envvars) (shader cache limits)
- [Steam's launch environment for app 730](https://gist.github.com/apprehensions/5d5f95347f69644ba7a6c93dab2a0173) (env dump) and [variables Steam sets for every game](https://github.com/ValveSoftware/Proton/issues/8926) (GitHub, Jul 2025)
- [NVIDIA shader cache merge bug, steam-for-linux #13622](https://github.com/ValveSoftware/steam-for-linux/issues/13622) (GitHub)
- [CS2 shader pre-caching on vs off](https://discuss.cachyos.org/t/cs2-triggers-1-5gb-2gb-updates-on-every-reboot-nvidia-shader-cache-persistence-issue-on-btrfs/32804) (CachyOS forum, Jul 2026) and [Steam background shader processing CPU use, #9266](https://github.com/ValveSoftware/steam-for-linux/issues/9266) (GitHub)
- [Building an Input Latency Meter](https://davidjusto.com/articles/m2p-latency/) (David Justo, Jan 2026: XWayland vs Wayland latency, `cs2.sh` logic)
- [fix-cs2-wayland](https://github.com/git-on-with-it/fix-cs2-wayland-linux) (stuck-mouse bug, CS2 display modes on Wayland, checked Nov 2026)
- [Going all-in on a Wayland future](https://blogs.kde.org/2025/11/26/going-all-in-on-a-wayland-future/) (KDE Blogs)
- [CachyOS Kernel Guide: Kernel Manager, Schedulers and Tuning](https://computingforgeeks.com/cachyos-kernel-manager-performance-tuning/) (ComputingForGeeks, Oct 2026)
- [Preparation steps: creating the USB drive](https://wiki.cachyos.org/installation/installation_prepare/) (CachyOS wiki)
- [CS2 FPS BENCHMARK DUST2](https://steamcommunity.com/sharedfiles/filedetails/?id=3240880604) (Steam Workshop) and [CS2 Linux Fixes](https://steamcommunity.com/sharedfiles/filedetails/?id=3728019787) (Steam guide, Jun 2026)
- [Best way to play Counter-Strike 2 stretched?](https://github.com/hyprwm/Hyprland/discussions/5685) (Hyprland discussion)
- [CachyOS August 2026 release notes](https://cachyos.org/blog/2608-august-release/) (update notifier)
- [Btrfs Snapshots](https://wiki.cachyos.org/configuration/btrfs_snapshots/) (CachyOS wiki, Jul 2026: default snapper limits and cleanup)
- [How to Install CachyOS: Step-by-Step](https://computingforgeeks.com/install-cachyos-step-by-step/) and [10 Best Linux Desktop Distributions 2026](https://computingforgeeks.com/best-linux-desktop-distributions/) (ComputingForGeeks)
- [Bazzite vs CachyOS 2026](https://tech-insider.org/bazzite-vs-cachyos-2026/) (Tech Insider)
- [AMD Officially Confirms The End Of The AMDVLK Driver](https://www.phoronix.com/news/AMDVLK-Discontinued) (Phoronix, 15 Sep 2025)
- [Arch Linux phasing out Pascal with NVIDIA 590](https://www.igorslab.de/en/arch-linux-is-phasing-out-pascal-gtx-10-cards-are-being-removed-from-the-default-path-with-nvidia-590/) (Igor'sLAB)
- [KDE Bug 487403: Allow Tearing on NVIDIA](https://bugs.kde.org/show_bug.cgi?id=487403) (KDE Bugzilla)
- [CS2 Input Lag (Wayland / XWayland / Windows / X11)](https://reddit.sentinel-team.org/posts/1r8vwo3/snapshots/2026-02-20T03%3A23%3A09.523908Z) (r/linux\_gaming snapshot, Feb 2026)
- [How to Show FPS in CS2](https://csdb.gg/guides/how-to-show-fps-cs2/) (csdb.gg)
- [Hyprland wiki: monitor modes and VRR](https://wiki.hypr.land/configuring/core/monitors/modes/) and [Tearing](https://wiki.hypr.land/Configuring/Advanced-and-Cool/Tearing/)
- [LACT 0.7.2 release notes](https://linuxcompatible.org/story/lact-072-released)
