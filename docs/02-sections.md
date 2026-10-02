# 2. Sections

> Part of the [catalog](../README.md). [Chapters](README.md) · [Bibliography](../references.md) · [Specimens](../specimens/README.md)
>
> ← [1. Introduction](01-introduction.md) · **2. Sections** · [3. Keys →](03-keys.md)

## Contents

- [2.1 Master sections table](#21-master-sections-table)
- [2.2 Core and legacy sections ([InternetShortcut], .A/.W variants, [DEFAULT], [DOC#n…])](#22-core-and-legacy-sections-internetshortcut-aw-variants-default-docn)
- [2.3 Feature sections ([Bookmarklet], [MonitoredItem])](#23-feature-sections-bookmarklet-monitoreditem)
- [2.4 Property-store (GUID) sections](#24-property-store-guid-sections)
- [2.5 The .website container](#25-the-website-container)
- [2.6 Parser edge cases](#26-parser-edge-cases)


A `.url` (Internet Shortcut) file is an INI-style text file: bracketed section headers followed by `Key=Value` lines, CR+LF line endings, historically ANSI-encoded.[^17][^18] The format has no published Microsoft specification for its on-disk layout; what is documented is the COM surface (`IUniformResourceLocator`, `IPersistFile`, `IPropertySetStorage`) that produces and consumes it.[^19] This chapter catalogs every section name observed in real `.url` and `.website` files, states which Windows component reads it and when it is written, and labels each entry with a confidence level: **[Documented]** (Microsoft documentation or SDK headers), **[Observed]** (seen in files written by Microsoft software or in-the-wild specimens), **[RE'd]** (established by reverse engineering), or **[Rumor]** (claimed but unverified).

Terminology used throughout: a **PROPID** is a numeric property identifier inside a property set; a **FMTID** is the GUID naming a property set; a **VT type tag** is the decimal OLE `VARTYPE` serialized before each property value (e.g. 31 = `VT_LPWSTR`, 19 = `VT_UI4`, 3 = `VT_I4`, 8 = `VT_BSTR`, 65 = `VT_BLOB`, 4127 = `VT_VECTOR|VT_LPWSTR`).[^20][^21]

## 2.1 Master sections table

| Section | Purpose | Read by | Written when | Confidence | Refs |
|---|---|---|---|---|---|
| `[InternetShortcut]` | Core shortcut: `URL`, `IconFile`, `IconIndex`, `HotKey`, `ShowCommand`, `Modified`, `WorkingDirectory`, `IDList`, `Roamed`, `Author`, `WhatsNew`, `Comment`, `Desc` | Shell `CInternetShortcut` (ieframe.dll / url.dll) via Win32 profile APIs; Explorer icon handler and property store at enumeration time | Every `.url`/`.website` save; only `URL=` is required | [Documented] (interface + MS sample; universally observed in the wild) | [^17][^19][^22] |
| `[InternetShortcut.A]` | ANSI (CP_ACP) shadow of the core section | Legacy ANSI profile readers | Written by IE/Explorer alongside `.W` when values are non-ASCII | [Observed] — `Procédure` specimen | [^17][^23] |
| `[InternetShortcut.W]` | UTF-7 shadow of the core section | Wide-character profile readers | Same trigger as `.A` | [Observed] — `Proc+AOk-dure` = `Procédure`; `+AF8-` = `_` in Blind Eagle sample | [^23][^24][^25] |
| `[DEFAULT]` | Frameset base URL (`BASEURL=`) | IE favorites restore logic | Saving a favorite of a framed page | [Observed] (real IE8 favorite; semantics per Blake's RE) | [^18][^26] |
| `[DOC#n(#n…)]` | Per-frame state: `BASEURL=`, `ORIGURL=`; nested frames append number pairs | IE favorites restore logic | Saving a favorite after navigating inside frames | [Observed] (Blake's extended-format specimen) | [^18][^26] |
| `[Bookmarklet]` | `ExtendedURL=` — full JavaScript bookmarklet (≤5119 bytes vs. 2083 for `URL=`) | IE favorites | Saving a `javascript:` favorite in IE | [Observed] | [^17][^27][^28] |
| `[MonitoredItem]` | `FeedUrl=`, `IsLivePreview=`, `PreviewSize=` — IE8 Web Slice / feed monitor | ieframe.dll (`MonitoredItem`, `FEEDURL="%s"` template strings present in the binary) | Subscribing to a Web Slice / feed monitor | [Observed] (Microsoft-shipped "Web Slice Gallery.url", "Suggested Sites.url") | [^17][^29][^30] |
| `[{000214A0-0000-0000-C000-000000000046}]` | FMTID_Intshcut property set; `PropN=` maps to legacy PROPID N | `IPropertySetStorage` on the Internet shortcut object; Explorer property system | Written by IE/Explorer on nearly every save (`Prop3=19,…` ubiquitous) | [Documented] (GUID + PID table in SDK headers; on-disk serialization observed) | [^19][^20][^21][^31] |
| `[{000214A1-0000-0000-C000-000000000046}]` | FMTID_InternetSite property set (PID_INTSITE_*: visit count, last visit, codepage, roaming, …) | `IPropertySetStorage` (documented API path) | **Never observed on disk** in any captured specimen; documented only as an API-accessible property set | [Documented] (GUID + PID table); on-disk presence unverified | [^19][^20][^21] |
| `[{9F4C2855-9F79-4B39-A8D0-E1D42DE1D5F3}]` | System property set: `Prop5=8,…` = System.AppUserModel.ID; `Prop31=` = VisualElementsManifest path | Explorer / taskbar pinning infrastructure | IE9+ pinned-site `.website`; documented MS 3D-app-launcher `.url` | [Documented] (PKEY_AppUserModel_ID; MS docs sample) | [^32][^33][^34] |
| `[{A7AF692E-098D-4C08-A225-D433CA835ED0}]` | `.website`-only pinned-site metadata (`Prop5=3,0`, `Prop9=19,0`, `Prop2=65,<blob>`, `Prop6=3,1`) | IE pinned-site infrastructure (presumed) | IE9+ pinned-site `.website` creation | [Observed]; GUID undocumented (not in ShlGuid.h/Propkey.h) | [^35][^36][^37] |
| `[{5CBF2787-48CF-4208-B90E-EE5E5D420294}]` | `Prop21=31,…` = System.Link.Description (Explorer "Description" field) | Explorer property system | Set via legacy Favorites property sheet | [Documented] (MS KB; also observed in specimens) | [^38][^39][^40] |
| `[{B9B4B3FC-2B51-4A42-B5D8-324146AFCF25}]` | `Prop5=31,…` = System.Link.Comment ("Notes" field) | Explorer property system | Same as above | [Documented] (MS KB; also observed in specimens) | [^38][^39][^40] |
| `[{64440492-4C8B-11D1-8B70-080036B11A03}]` | `Prop9=19,{1,25,50,75,99}` = System.Rating (1–5 stars) | Explorer property system | Same as above | [Documented] (MS KB; also observed in specimens) | [^38][^39][^41] |
| `[{F29F85E0-4FF9-1068-AB91-08002B27B3D9}]` | FMTID_SummaryInformation: `Prop2=31` Title, `Prop3=31` Subject, `Prop4=4127` Author, `Prop5=4127` Keywords, `Prop6=31` Comment | Explorer property system | Observed in a user-dumped real `.url` (MS Q&A) | [Observed] (MS Q&A specimen); GUID itself is the standard documented SummaryInformation FMTID | [^40][^41] |

Two notes on the master table. First, the `[{000214A1-…}]` (FMTID_InternetSite) row is the only entry that is fully documented yet never observed: Microsoft documents 20 PROPIDs for it (visit count, last-visit FILETIME, codepage, roaming state, and a `PID_INTSITE_RAWURL` that appears in the ShlObj.h comment block with no `#define`), but no captured `.url` or `.website` specimen contains the section.[^19][^20] It is cataloged here because it is the documented sibling of FMTID_Intshcut and any compliant reader must tolerate it. Second, source conflicts exist on peripheral claims: filetypedb.com lists `Referer=`, `BrowserFlags=`, and `BaseURL=`-inside-`[InternetShortcut]` as format fields, but no specimen, SDK property ID, or other source corroborates any of them; they are classified as rumors (see [§2.6](#26-parser-edge-cases) and [the rumor table](07-rumors-versions-gaps.md#61-unverified-and-rumored-keys) of this catalog).[^42]

## 2.2 Core and legacy sections ([InternetShortcut], .A/.W variants, [DEFAULT], [DOC#n…])

`[InternetShortcut]` is the only mandatory section, and within it only `URL=` is required.[^17] The full key set corroborated across the NSIS wiki, Blake's Unofficial Guide, format-aggregator sites, and real specimens is: `URL`, `HotKey`, `IconFile`, `IconIndex`, `ShowCommand`, `Modified`, `WorkingDirectory`, `Roamed`, `IDList`, `Author`, `WhatsNew`, `Comment`, `Desc`.[^17][^18][^43] The legacy keys `Author`/`WhatsNew`/`Comment`/`Desc` are IE4-era favorites metadata (they map to PID_IS_AUTHOR/WHATSNEW/COMMENT/DESCRIPTION in FMTID_Intshcut) and are no longer surfaced by the modern shell UI, though an ExifTool test-suite specimen exercises all of them.[^20][^43]

The `.A` and `.W` variants were guessed on the NSIS wiki ("CP_ACP stuff?" / "UTF-7 stuff?") and are now confirmed by a genuine user file posted to Microsoft Q&A, in which the same target appears three ways:[^17][^23]

```
[InternetShortcut]
URL=file://server-01/share/home/msmith/Procédure Signature.doc
[InternetShortcut.A]
URL=file://server-01/share/home/msmith/Procédure Signature.doc
[InternetShortcut.W]
URL=file://server-01/share/home/msmith/Proc+AOk-dure Signature.doc
```

`Proc+AOk-dure` is the UTF-7 encoding of `Procédure`, while the `.A` copy holds the raw ANSI byte for `é`. A Japanese reverse-engineering write-up independently describes the `.W` encoding as modified UTF-7 (non-ASCII runs Base64-encoded between `+` and `-`, with `+`→`+-` and `-`→`-+` escaping), and the Blind Eagle in-the-wild specimen shows `+AF8-` (UTF-7 for `_`) inside the `.W`-section URL.[^25][^24] GUID property sections can also carry `.A`/`.W` shadows, e.g. `[{5CBF2787-…}.W] Prop21=31,System.Link.Description+iqxmDg-`.[^40]

The frameset sections preserve per-frame navigation state. `[DEFAULT]` holds the frameset `BASEURL=`; each `[DOC#n(#n…)]` section holds one frame's `BASEURL=` (absolute) and `ORIGURL=` (as-loaded, possibly relative), and nested frames append further number pairs (`[DOC#4#5#4#6]` is a frame nested inside `[DOC#4#5]`).[^18] Blake's 3rd-edition specimen:[^18]

```
[DEFAULT]
BASEURL=http://www.someaddress.com

[DOC#4#5]
BASEURL=http://www.someaddress.com/frame1.html
ORIGURL=frame1.html

[DOC#4#6]
BASEURL=http://www.someaddress.com/frame2.html
ORIGURL=frame2.html

[InternetShortcut]
URL=http://www.someaddress.com/
```

Real IE8-generated favorites confirm the `[DEFAULT] BASEURL=` placement; `BASEURL=` inside `[InternetShortcut]` has no real-Windows evidence and is a rumor propagated by one aggregator site.[^26][^42]

## 2.3 Feature sections ([Bookmarklet], [MonitoredItem])

| Section | Keys | Limits / semantics | Confidence | Refs |
|---|---|---|---|---|
| `[Bookmarklet]` | `ExtendedURL=` | Full bookmarklet script, ≤5119 bytes (IE11); `URL=` holds a 2083-byte truncation and must be kept in sync or the bookmarklet breaks; tooltip truncated to 255 bytes | [Observed] | [^17][^27][^28] |
| `[MonitoredItem]` | `FeedUrl=`, `IsLivePreview=`, `PreviewSize=` | IE8 Web Slice / feed-monitor favorite; `PreviewSize=320x240` observed; `FeedUrl` spelling (lowercase "rl") consistent across specimens | [Observed] | [^29][^30][^44] |

Bookmarklet specimen (GitHub gist, verbatim):[^27]

```
[{000214A0-0000-0000-C000-000000000046}]
Prop3=19,15
[InternetShortcut]
URL=javascript:alert('oh yea')
IDList=
[Bookmarklet]
ExtendedURL=javascript:alert('oh yea')
```

A bookmarklet generator blog documents that when the script contains CJK characters the file must be saved as UTF-16LE ("unicode"), not UTF-8 — an encoding edge case revisited in [§2.6](#26-parser-edge-cases).[^28] For `[MonitoredItem]`, Microsoft-shipped Web Slice files recovered in sandbox analyses contain `[MonitoredItem] FeedUrl=https://ieonline.microsoft.com/#ieslice … PreviewSize=320x240 IsLivePreview=true`, and ieframe.dll itself embeds the template strings `FEEDURL="%s"` and `MonitoredItem`, evidencing that the shell/IE — not third parties — writes this section.[^29][^30]

## 2.4 Property-store (GUID) sections

Any section whose name is a braced GUID serializes one COM property set: each `PropN=<vt>,<value>` line stores PROPID **N** with OLE VARTYPE **vt**.[^20][^21] Microsoft documents the COM access path (`IPropertySetStorage::Open` with FMTID_Intshcut or FMTID_InternetSite) but not the INI serialization; the one official on-disk sample is in the Mixed Reality 3D-app-launcher documentation.[^19][^34]

| GUID section | Identity | Observed keys | Meaning | Confidence | Refs |
|---|---|---|---|---|---|
| `{000214A0-0000-0000-C000-000000000046}` | FMTID_Intshcut | `Prop3=19,{0\|2\|9\|11\|15}`; `Prop4=31,<title>` | Prop3 = undocumented PROPID 3 (the public PID_IS_* list skips 2→4); semantics unknown — values 0/2/9/11/15 all observed, community sources explicitly admit the meaning is unknown. Prop4 = PID_IS_NAME (shortcut title) | [Documented] (GUID/PIDs); Prop3 semantics observed-only, unknown | [^20][^21][^31][^35] |
| `{000214A1-0000-0000-C000-000000000046}` | FMTID_InternetSite | (none observed) | Documented API property set for site metadata (visits, roaming, codepage); never seen serialized on disk | [Documented]; on-disk presence unverified | [^19][^20][^21] |
| `{9F4C2855-9F79-4B39-A8D0-E1D42DE1D5F3}` | System property set | `Prop5=8,Microsoft.Website.<8hex>.<8hex>`; `Prop31=<path>; Prop5=ExplicitAppUserModelID` (MS launcher sample); `Prop12=19,2` (one specimen) | Prop5 = System.AppUserModel.ID (documented PKEY, propID 5); the `Microsoft.Website.*` AUMID naming convention itself is undocumented | [Documented] (PKEY + MS sample); AUMID naming convention observed-only | [^32][^33][^34][^36] |
| `{A7AF692E-098D-4C08-A225-D433CA835ED0}` | (undocumented) | `Prop5=3,0`; `Prop9=19,0`; `Prop2=65,<hex blob>`; `Prop6=3,1` | `.website`-only pinned-site metadata; Prop2 blob semantics undocumented; modern (2022) files omit Prop2/Prop6 | [Observed] | [^35][^36][^37] |
| `{5CBF2787-48CF-4208-B90E-EE5E5D420294}` | System.Link.* set | `Prop21=31,<text>` (+`.A`/`.W` shadows) | System.Link.Description — Explorer "Description" field | [Documented] (MS KB) | [^38][^39][^40] |
| `{B9B4B3FC-2B51-4A42-B5D8-324146AFCF25}` | System.Link.Comment set | `Prop5=31,<text>` | "Notes" field | [Documented] (MS KB) | [^38][^39][^40] |
| `{64440492-4C8B-11D1-8B70-080036B11A03}` | PSGUID_MEDIAFILESUMMARYINFORMATION | `Prop9=19,{1,25,50,75,99}` | System.Rating: 1/25/50/75/99 = 1–5 stars | [Documented] (MS KB) | [^38][^39][^41] |
| `{F29F85E0-4FF9-1068-AB91-08002B27B3D9}` | FMTID_SummaryInformation | `Prop2=31` Title, `Prop3=31` Subject, `Prop4=4127` Author, `Prop5=4127` Keywords, `Prop6=31` Comment | Standard document-summary properties carried inside a `.url` | [Observed] (MS Q&A dump) | [^40][^41] |

The three extended-favorite sections (Description, Notes, Rating) are unusual in being documented by Microsoft at the byte level: a Microsoft support KB ("apply-property-error") gives the exact section GUIDs, Prop numbers, type tags, and star-rating values, and the metabpa bookmark-format project explicitly cites that KB as its source.[^38][^39] The `Prop3` row is the catalog's main open conflict: Quarkslab's CVE-2016-3353 analysis assigns it no exploit role, benign files carry 0/2/11, malware in the CVE-2024-43451 campaign carries 9 under a look-alike GUID (`{009862A0-…}` in place of `{000214A0-…}`; the PoC variant additionally malforms the fourth group to `C0000`), and no source explains any value — this catalog therefore records it as "undocumented PROPID 3, VT_UI4; semantics unknown" rather than asserting a meaning.[^31][^24][^51]

## 2.5 The .website container

The `.website` extension is IE9+'s pinned-site format. Microsoft's only functional description: "a .website file is like a shortcut, except it's a plain text file that describes not only the website's URL but also how the icon looks"; no byte-level spec was ever published.[^33] On disk it is a `.url` file plus three additions: `Prop4=31,<title>` in the FMTID_Intshcut section, the undocumented `{A7AF692E-…}` section, and the `{9F4C2855-…}` section carrying the auto-generated AppUserModelID.[^35][^36] Canonical specimen (google.com, forum-documented):[^37]

```
[{000214A0-0000-0000-C000-000000000046}]
Prop3=19,2
Prop4=31,Google
[InternetShortcut]
IDList=
URL=https://www.google.com
IconFile=https://www.google.com/favicon.ico
IconIndex=1
[{A7AF692E-098D-4C08-A225-D433CA835ED0}]
Prop5=3,0
Prop9=19,0
Prop2=65,2C0000000000000001000000FFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFF750100004000000025060000690300006A
Prop6=3,1
[{9F4C2855-9F79-4B39-A8D0-E1D42DE1D5F3}]
Prop5=8,Microsoft.Website.A7CF2A08.64C3029D
```

Specimens show only standard `[InternetShortcut]` keys plus `Prop*` lines — no named keys beyond `Prop*` exist in any captured `.website`, busting that rumor.[^36] Behaviorally, `.website` files open in IE (ProgID invokes `iexplore.exe -w "%l" %*`, an undocumented switch) rather than the default browser, and the extension is hidden by `NeverShowExt` just like `.url`.[^45] The format is attack-relevant: the CVE-2025-24054 in-the-wild campaign used a `.website` with `URL=file://<ip>/` and all three GUID sections intact to coerce NTLM authentication.[^35]

## 2.6 Parser edge cases

The shell parses `.url` files with the Win32 private-profile APIs (`GetPrivateProfileString` et al.), so INI-parser semantics — not any URL grammar — define what is accepted.[^18][^46] The Rust crate `rclip-url-file`, written against Blake's guide and Wine's `dlls/ieframe/intshcut.c` reimplementation, documents these behaviors:[^46][^47]

| Behavior | Rule | Consequence | Confidence | Refs |
|---|---|---|---|---|
| Case | Section and key names compare ASCII-case-insensitively | Wine writes `ICONFILE=`/`ICONINDEX=` and reads them back as `iconfile`/`iconindex`; a case-sensitive parser silently loses the icon | [RE'd] (Wine source) | [^47] |
| Whitespace | Whitespace around `=` and around the value is stripped | `URL = http://x` parses as `URL=http://x` | [RE'd] | [^47] |
| Quotes | A matched pair of double quotes around the value is stripped | Installers rely on this to preserve trailing spaces | [RE'd] (GetPrivateProfileString semantics) | [^47] |
| Duplicates | First occurrence of a key wins; first section of a given name wins | Later duplicate keys/sections are dead text | [RE'd] | [^47] |
| Comments | `;` starts a comment; `#` does not | A leading `#` is part of the line | [RE'd] (Win32 profile API) | [^47] |
| Section order | Not fixed | Real Explorer-written file has order `000214A0 → A7AF692E → InternetShortcut → 9F4C2855` — `[InternetShortcut]` is **not** always first | [Observed] (SO 62490091 specimen) | [^48] |
| Encoding | Legacy files are ANSI/CP_ACP; Wine writes UTF-8; UTF-16LE with BOM occurs (bookmarklet generator; YARA `wide` rules); BOM stripped by tolerant parsers | A legacy code-page file must be transcoded before UTF-8-only parsing | [Observed] (encodings seen in the wild; parser handling per Wine/rclip RE) | [^28][^46][^49] |

Malware actively exploits parser differentials around these rules. The CVE-2024-43451 PoC and the Blind Eagle campaign both ship files where `[InternetShortcut.A]` and `[InternetShortcut.W]` are present but *empty*, with the operative `URL=` placed after them — so a naive parser that reads only the first `URL=` (or only the `[InternetShortcut]` section) reports a different target than the shell resolves.[^24][^51] The PoC specimen further adds bidi Unicode and malformed GUID sections (`C0000` for `C000`) that match no real property set.[^51] Detection rules accordingly tolerate leading/inner whitespace in the section header (`/^[ \t]*\[[ \t]*InternetShortcut[ \t]*\]/i`) and UTF-16LE files.[^49] Finally, for completeness: `Referer=`, `BrowserFlags=`, `SiteURL=`, `ScriptUrl=`, pinned/taskbar keys, and `BaseURL=` inside `[InternetShortcut]` are all **[Rumor]** — no specimen, SDK property ID, or code supports them; `BrowserFlags` is a real HKCR registry value that one aggregator site appears to have cross-wired into the file format.[^42][^50]

---

## Footnotes

[^17]: https://nsis.sourceforge.io/Creating_internet_shortcuts — NSIS wiki, "Creating internet shortcuts" (full section/key map; .A/.W guesses; 2083/5119 limits)
[^18]: http://www.cyanwerks.com/formats/file-format-url.html — Edward L. Blake, "An Unofficial Guide to the URL File Format", 3rd ed. (CRLF/ANSI, profile-API manipulation, frameset [DEFAULT]/[DOC#n] extended format)
[^19]: https://learn.microsoft.com/en-us/windows/win32/lwef/internet-shortcuts — Microsoft Learn, "Internet Shortcuts" (IUniformResourceLocator, IPropertySetStorage, FMTID_Intshcut / FMTID_InternetSite PID tables)
[^20]: https://github.com/tpn/winsdk-10/blob/master/Include/10.0.10240.0/um/ShlObj.h — Windows 10 SDK ShlObj.h (PID_IS_* and PID_INTSITE_* numeric PROPIDs)
[^21]: https://github.com/tpn/winsdk-10/blob/master/Include/10.0.10240.0/um/ShlGuid.h — Windows 10 SDK ShlGuid.h (FMTID_Intshcut / FMTID_InternetSite GUID definitions)
[^22]: https://learn.microsoft.com/en-us/windows/mixed-reality/distribute/implementing-3d-app-launchers-win32 — Microsoft Learn, "Implementing 3D app launchers" (official sample .URL with GUID sections)
[^23]: https://learn.microsoft.com/en-us/answers/questions/2150484/extremely-slow-open-file-dialog-from-all-applicati — Microsoft Q&A 2150484 (Procédure / Proc+AOk-dure .A/.W specimen)
[^24]: https://research.checkpoint.com/2025/blind-eagle-and-justice-for-all/ — Check Point Research, "Blind Eagle" (in-the-wild .A/.W parser-differential specimen, +AF8- UTF-7)
[^25]: https://www.tatsu-syo.info/Devroom/IEfavorites.html — tatsu-syo.info (Japanese RE page: .W = modified UTF-7, +→+- / -→-+ escaping)
[^26]: https://ubuntugenius.wordpress.com/2009/12/09/how-to-open-url-internet-explorer-shortcuts-in-ubuntu-using-firefox/ — real IE8-generated favorite with [DEFAULT] BASEURL= specimen
[^27]: https://gist.github.com/dungsaga/45788260c81832de54ab5e237b523d22 — GitHub gist: IE bookmarklet .url specimen and 5119/2083/255-byte limits
[^28]: https://blog.darkthread.net/blog/ie-bookmarklet/ — darkthread blog: bookmarklet generator; UTF-16LE requirement for CJK; URL/ExtendedURL sync
[^29]: https://hybrid-analysis.com/sample/db1696106bb100a1fc10fadc9b93e17f80055604fa53fe3785b64efdabe0a254/56af7da60e316d585ed41a73 — Hybrid Analysis: Microsoft-shipped "Web Slice Gallery.url" / "Suggested Sites.url" strings
[^30]: https://community.sap.com/t5/application-development-discussions/sending-url-as-attachment/m-p/9339383 — SAP Community: [MonitoredItem] FeedUrl/IsLivePreview specimen
[^31]: https://blog.quarkslab.com/analysis-of-ms16-104-url-files-security-feature-bypass-cve-2016-3353.html — Quarkslab, CVE-2016-3353 analysis (ieframe OpenURL; no exploit role for Prop3)
[^32]: https://learn.microsoft.com/en-us/windows/win32/properties/props-system-appusermodel-id — Microsoft Learn, System.AppUserModel.ID (formatID 9F4C2855-…, propID 5)
[^33]: https://learn.microsoft.com/en-us/previous-versions/windows/internet-explorer/ie-it-pro/internet-explorer-11/ie11-deploy-guide/deploy-pinned-sites-using-mdt-2013 — Microsoft Learn, IE11 deployment guide ("a .website file is like a shortcut…")
[^34]: https://github.com/MicrosoftDocs/mixed-reality/blob/docs/mixed-reality-docs/mr-dev-docs/distribute/implementing-3d-app-launchers-win32.md — MicrosoftDocs/mixed-reality repo (sample .URL launcher with {9F4C2855} Prop31/Prop5)
[^35]: https://research.checkpoint.com/2025/cve-2025-24054-ntlm-exploit-in-the-wild/ — Check Point Research, CVE-2025-24054 (malicious .website with all three GUID sections)
[^36]: https://www.autoitscript.com/forum/topic/150926-internet-shortcut-sanitizer-url-and-website-files/ — AutoIt forum: full real .website specimen (Prop12 variant; Prop*-only keys)
[^37]: https://community.spiceworks.com/t/microsoft-edge-desktop-shortcut/790553 — Spiceworks Community: canonical google.com .website specimen
[^38]: https://learn.microsoft.com/zh-cn/previous-versions/troubleshoot/browsers/core-features/apply-property-error — Microsoft Support KB "apply-property-error" (Description/Notes/Rating section GUIDs, Prop numbers, star values)
[^39]: http://www.metabpa.org/projects/psbrowserbookmarks/about_browserbookmarks — metabpa.org browser bookmarks project (GUID sections and Prop meanings; cites the MS KB)
[^40]: https://learn.microsoft.com/ja-jp/answers/questions/4115480/question-4115480 — Microsoft Q&A (ja): full property-set dump incl. {F29F85E0} SummaryInformation and .A/.W GUID-section shadows
[^41]: https://www.voidtools.com/support/everything/properties/ — voidtools Everything docs (property-set serialization syntax, {F29F85E0} and {64440492} examples)
[^42]: https://filetypedb.com/web/url — filetypedb.com "URL File Format" (weak/uncorroborated aggregator; source of Referer/BrowserFlags/BaseURL rumors)
[^43]: https://raw.githubusercontent.com/exiftool/exiftool/master/t/images/LNK.url — ExifTool test-suite .url specimen exercising Author/WhatsNew/Comment/Desc/Roamed/IDList
[^44]: http://forum.hotfix.pl/problemy/problem-ze-skrotem-uslugi-sugerowane-witryny-w-ie-t21332.html — hotfix.pl forum: full "Suggested Sites.url" specimen with PreviewSize=320x240
[^45]: https://www.zdnet.com/article/ie9-power-tips-the-secrets-of-pinned-site-shortcuts/ — ZDNet (Ed Bott): .website launches `iexplore.exe -w "%l" %*`
[^46]: https://docs.rs/rclip-url-file/latest/rclip_url_file/ — rclip-url-file crate docs (no-specification statement; UTF-8/BOM handling; Wine basis)
[^47]: https://docs.rs/rclip-url-file/latest/rclip_url_file/ini/index.html — rclip-url-file ini module (case-insensitivity, whitespace/quote stripping, first-duplicate-wins, ';'-only comments)
[^48]: https://stackoverflow.com/questions/62490091 — Stack Overflow 62490091: real Explorer-written file with [InternetShortcut] not first
[^49]: https://tharros.com/the-dangers-of-windows-internetshortcut-url-files/ — tharros.com (Will Dormann YARA rules: whitespace-tolerant header, wide/UTF-16 variant)
[^50]: https://blog.51cto.com/qicaiiwang/432259 — 51cto blog: BrowserFlags as HKCR registry value (registry/file-format confusion)
[^51]: https://gist.github.com/milo2012/00856e9273ab08829dc715a845abb4ed — GitHub gist: CVE-2024-43451 PoC .url (empty .A/.W sections, bidi Unicode, malformed C0000 GUID sections, Prop3=19,9)

<!-- chapter-nav -->

---

← [1. Introduction](01-introduction.md) · [Chapters](README.md) · [3. Keys →](03-keys.md)
