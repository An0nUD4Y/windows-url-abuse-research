# 1. Introduction, Scope & Method

> Part of the [catalog](../README.md). [Chapters](README.md) · [Bibliography](../references.md) · [Specimens](../specimens/README.md)
>
> **1. Introduction** · [2. Sections →](02-sections.md)

## Contents

- [1.1 The .url file and its undocumented status](#11-the-url-file-and-its-undocumented-status)
- [1.2 Who actually parses .url files](#12-who-actually-parses-url-files)
- [1.3 Evidence sources and confidence model](#13-evidence-sources-and-confidence-model)


## 1.1 The .url file and its undocumented status

A Windows Internet Shortcut (`.url`) is, on disk, an INI-style text file: a `[InternetShortcut]` section header, `key=value` lines, CRLF line terminators, and ANSI (system code page) character encoding in the legacy form.[^1] It can be read and written with the Win32 profile APIs (`GetPrivateProfileString` / `WritePrivateProfileString`), which is why its parsing inherits INI semantics — ASCII-case-insensitive section and key names, whitespace trimming, matched-double-quote stripping, and first-duplicate-wins.[^2] Microsoft has never published a file-format specification for it; the most complete unofficial key list, maintained on the NSIS wiki, states plainly: "The .URL file format is not officially documented."[^3] The Rust crate `rclip-url-file`, itself a third-party parser, opens its documentation with the same caveat: "There is no specification. Microsoft never published one."[^2]

What Microsoft *did* document is a COM surface, not a byte layout. The Internet shortcut object (`CLSID_InternetShortcut` = {FBF23B40-E3F0-101B-8488-00AA003E56F8}) is created with `CoCreateInstance`, its target set through `IUniformResourceLocator::SetURL`, and the file written with `IPersistFile::Save`.[^4] The object also exposes `IPropertySetStorage`, through which callers open one of two property sets identified by FMTIDs (format IDs — GUIDs that name a property set): `FMTID_Intshcut` = {000214A0-0000-0000-C000-000000000046} and `FMTID_InternetSite` = {000214A1-0000-0000-C000-000000000046}, both defined in the Windows SDK header `ShlGuid.h`.[^5] The documented properties (`PID_IS_URL`, `PID_IS_ICONFILE`, and so on, with numeric PROPIDs in `ShlObj.h`) are enumerated on the Microsoft Learn "Internet Shortcuts" page, now hosted under Legacy Windows Environment Features.[^4][^6]

The critical gap — and the reason this catalog exists — is that the documented COM surface is a *subset* of what the on-disk parser honors. Real `.url` files written by Explorer are serialized property stores: `[{000214A0-…}]` sections containing `Prop<N>=<VARTYPE>,<value>` lines, where N is a PROPID and the leading number is a COM variant type tag (VT tag).[^6] These sections carry *undocumented* PROPIDs — most prominently `Prop3`, which appears in benign files (`Prop3=19,11`), in vendor files (`Prop3=19,0`), and in exploit specimens (`Prop3=19,9`) — alongside hand-authored INI keys like `WorkingDirectory=` that the property API also exposes.[^7] Microsoft's only published *sample file* in this INI serialization is a launcher shortcut in the Mixed Reality documentation; a support KB separately documents three extended-property sections at the byte level (see [§2.4](02-sections.md#24-property-store-guid-sections)).[^8] [Chapter 2](02-sections.md) catalogs the sections, [Chapter 3](03-keys.md) the keys, and [Chapter 4](04-property-bags.md) decodes the property-bag serialization in full.

## 1.2 Who actually parses .url files

Registry wiring determines the open-time path. `HKCR\.url` maps the extension to the ProgID `InternetShortcut` (on Windows 8+ also `IE.AssocFile.URL`), whose `NeverShowExt` value hides the extension in Explorer even when "hide extensions" is disabled.[^9][^10] The `shell\open\command` value has changed exactly once since the XP era: `rundll32.exe shdocvw.dll,OpenURL %l` on pre-SP3 XP, replaced by `rundll32.exe ieframe.dll,OpenURL %l` from XP SP3 through Windows 11 — verified in registry dumps from live Windows 11 systems.[^11][^12] Quarkslab's CVE-2016-3353 analysis traces `ieframe!OpenURL` into the `CInternetShortcut` class, whose execute helper ultimately calls `ShellExecuteEx`; after MS16-104, files carrying a Mark of the Web (MotW — the `Zone.Identifier` alternate data stream recording a file's origin zone) are gated by a security dialog.[^11]

Equally important is what happens *without* an open. Explorer parses `.url` files at enumeration time — to resolve `IconFile`/`IconIndex`, the `IDList`, and the property store — whenever a folder is merely displayed, right-clicked, or dragged.[^9] Behavioral evidence is unambiguous: a Microsoft Q&A thread documents every application's Open File dialog hanging on a folder containing `.url` files with `URL=file://server/…` targets, and Check Point's CVE-2024-43451 write-up lists "a single right-click on the file; deleting the file; dragging the file to another folder" as triggers.[^13][^14] This report therefore distinguishes **render-time parsing** ([Chapter 5](05-security-behaviors.md)'s NTLM-leak class) from **open-time parsing** (the execution-context class) throughout; defenses that scan only at open miss the entire first class.

Finally, the open-source reimplementations prove where the real parser lives. Whole-tree analysis of Wine and ReactOS shows their `ieframe/intshcut.c` reads exactly one section and three keys — `[InternetShortcut]`, `URL`, `iconfile`, `iconindex` — and Wine's own test fixture asserts that a `[{000214A0-…}]` section with `Prop0=1,2` is *ignored*.[^15][^16] Neither tree contains any `Modified=`, `HotKey`, `WorkingDirectory`, or `Prop<N>` handling.[^15] The full parser exists only in Microsoft's `ieframe.dll`/`shell32.dll`, which is why every non-trivial row in this catalog rests on reverse engineering or specimens rather than on documentation.

## 1.3 Evidence sources and confidence model

Evidence was gathered from five source classes, in descending priority: (1) Microsoft Learn pages and Windows SDK headers; (2) primary vendor research (Check Point, Trend Micro, Quarkslab, ZDI, 0patch); (3) binary reimplementations (Wine, ReactOS) and their test suites; (4) verbatim in-the-wild and Microsoft-written file specimens; (5) community documentation (NSIS wiki, Blake's *Unofficial Guide*, format-aggregator sites — the last flagged as weak). Every catalog row and every standalone claim about format semantics carries exactly one confidence label:

| Label | Meaning | Example used in this report |
|---|---|---|
| **[Documented]** | Appears in Microsoft documentation or SDK headers | `PID_IS_ICONFILE` = PROPID 9, VT_LPWSTR, in `ShlObj.h`[^6] |
| **[Observed]** | Seen in real files written by Microsoft software or in-the-wild specimens; specimen cited | `Prop3=19,11` in an Explorer-written GitHub shortcut[^7] |
| **[RE'd]** | Established by binary analysis or Wine/ReactOS source; code cited | Case-insensitive key lookup via `GetPrivateProfileStringW` in Wine `intshcut.c`[^15] |
| **[Rumor]** | Claimed but unverified; confined to the rumor table ([Chapter 6](07-rumors-versions-gaps.md)) | `Referer=` / `BrowserFlags=` as `.url` keys (no specimen, contradicted by other sources) |

Two methodological rules follow. First, negative results are reported as such: when a key appears in no specimen, no SDK header, and no parser, it is a rumor, not a deprecated feature. Second, because Microsoft's security fixes have progressively *depowered* individual fields (the MS16-104 MotW gate, the CVE-2025-33053 `WorkingDirectory` fix) rather than document the format, field semantics are version-dependent and are stated per-field, per-version in [Chapter 3](03-keys.md) and [Chapter 5](05-security-behaviors.md).[^11]

---

## Footnotes

[^1]: http://www.cyanwerks.com/formats/file-format-url.html — Edward L. Blake, "An Unofficial Guide to the URL File Format" (CRLF + ANSI layout)
[^2]: https://docs.rs/rclip-url-file/latest/rclip_url_file/ — rclip-url-file crate docs: no specification; INI parsing edge cases
[^3]: https://nsis.sourceforge.io/Creating_internet_shortcuts — NSIS wiki, "Creating internet shortcuts" (unofficial key/section list)
[^4]: https://learn.microsoft.com/en-us/windows/win32/lwef/internet-shortcuts — Microsoft Learn, "Internet Shortcuts" (CLSID_InternetShortcut, IUniformResourceLocator, IPropertySetStorage usage)
[^5]: https://github.com/tpn/winsdk-10/blob/master/Include/10.0.10240.0/um/ShlGuid.h — Windows 10 SDK ShlGuid.h (FMTID_Intshcut / FMTID_InternetSite GUID definitions)
[^6]: https://github.com/tpn/winsdk-10/blob/master/Include/10.0.10240.0/um/ShlObj.h — Windows 10 SDK ShlObj.h (PID_IS_* numeric PROPIDs and variant types)
[^7]: https://github.com/jx-admin/Code2/blob/master/androidCode.url — committed .url specimen with `[{000214A0-…}] Prop3=19,11`
[^8]: https://learn.microsoft.com/en-us/windows/mixed-reality/distribute/implementing-3d-app-launchers-win32 — Microsoft Learn, sample .URL launcher shortcut (only MS-published INI serialization example)
[^9]: https://www.nccgroup.com/research/the-case-of-missing-file-extensions/ — NCC Group, "The Case of Missing File Extensions" (NeverShowExt list includes .URL and .website)
[^10]: https://github.com/MakiseKurisu/Win86emu/blob/master/yact/_ReactOS_Dlls/x86node.reg — registry dump of HKCR\InternetShortcut (NeverShowExt, IsShortcut, shdocvw OpenURL command)
[^11]: https://blog.quarkslab.com/analysis-of-ms16-104-url-files-security-feature-bypass-cve-2016-3353.html — Quarkslab, MS16-104 / CVE-2016-3353 analysis (ieframe!OpenURL → CInternetShortcut; MotW dialog gate)
[^12]: https://www.tenforums.com/general-support/193229-windows-file-explorer-wont-execute-url-shortcuts-3.html — Windows 11 registry dump: `rundll32.exe ieframe.dll,OpenURL %l`; XP→SP3 shdocvw swap corroborated at https://stackoverflow.com/questions/30670975
[^13]: https://learn.microsoft.com/en-us/answers/questions/2150484/extremely-slow-open-file-dialog-from-all-applicati — Microsoft Q&A: Open File dialogs hang on folders containing .url files (render-time parsing evidence)
[^14]: https://research.checkpoint.com/2025/blind-eagle-and-justice-for-all/ — Check Point Research, Blind Eagle / CVE-2024-43451 (right-click, delete, drag as triggers)
[^15]: https://github.com/wine-mirror/wine/blob/master/dlls/ieframe/intshcut.c — Wine ieframe intshcut.c (reads only URL/iconfile/iconindex; GetPrivateProfileStringW semantics)
[^16]: https://github.com/wine-mirror/wine/blob/master/dlls/ieframe/tests/intshcut.c — Wine test fixture asserting `[{000214A0-…}] Prop0=1,2` is ignored; ReactOS equivalent: https://github.com/reactos/reactos/blob/master/dll/win32/ieframe/intshcut.c

<!-- chapter-nav -->

---

[Chapters](README.md) · [2. Sections →](02-sections.md)
