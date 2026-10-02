# 4. Property-Bag Decoding

> Part of the [catalog](../README.md). [Chapters](README.md) · [Bibliography](../references.md) · [Specimens](../specimens/README.md)
>
> ← [3. Keys](03-keys.md) · **4. Property bags** · [5. Security behaviors →](05-security-behaviors.md)

## Contents

- [4.1 The Prop<N>=<vt>,<value> grammar](#41-the-propnvtvalue-grammar)
- [4.2 VT type tags observed on disk](#42-vt-type-tags-observed-on-disk)
- [4.3 FMTID_Intshcut [{000214A0-…}]: the documented PID_IS_* table](#43-fmtid_intshcut-000214a0--the-documented-pid_is_-table)
- [4.4 FMTID_InternetSite [{000214A1-…}]: the documented PID_INTSITE_* table](#44-fmtid_internetsite-000214a1--the-documented-pid_intsite_-table)
- [4.5 Observed-but-undocumented property IDs](#45-observed-but-undocumented-property-ids)
- [4.6 The .website property sections](#46-the-website-property-sections)
- [4.7 Confirmed vs inferred: mapping summary](#47-confirmed-vs-inferred-mapping-summary)


Beyond the flat keys of `[InternetShortcut]`, .url and .website files carry a second, typed data layer: GUID-named INI sections that serialize a COM property store (the `IPropertySetStorage`/`IPropertyStorage` view of the Internet Shortcut object).[^84][^85] Microsoft documents the property sets and their property IDs (PROPIDs) at the API level, but has never published the on-disk `Prop<N>=` grammar; [§4.1](#41-the-propnvtvalue-grammar), [§4.2](#42-vt-type-tags-observed-on-disk), [§4.5](#45-observed-but-undocumented-property-ids) and [§4.6](#46-the-website-property-sections) are reconstructed from real files and cross-checked against the SDK headers.

## 4.1 The Prop<N>=<vt>,<value> grammar

Each GUID-named section `[{XXXXXXXX-XXXX-XXXX-XXXX-XXXXXXXXXXXX}]` serializes one property set, where the GUID is the set's FMTID (format ID). Inside the section, each line has the form:

```
Prop<N>=<vttype>,<value>
```

- `N` is the legacy PROPID within that property set. The NSIS wiki's section map gives the range as `Prop<2..2147483647>` — i.e. any PROPID from 2 to 2³¹−1 is syntactically legal, and the wiki explicitly labels the `{000214A0-…}` section "FMTID_Intshcut property storage".[^84]
- `<vttype>` is the OLE VARTYPE tag written as a decimal integer, followed by a comma.
- `<value>` is the serialized property value: literal text for string types, a decimal integer for numeric types, lowercase hex for blobs.

The grammar is observed, not documented. The strongest corroboration that `N` really is the PROPID is self-consistency: `Prop4=31,<title>` in FMTID_Intshcut matches the documented `PID_IS_NAME = 4` (VT_LPWSTR = 31),[^85][^86] and `Prop5=8,Microsoft.Website.…` in `{9F4C2855-…}` matches the documented `System.AppUserModel.ID` property key (formatID `9F4C2855-9F79-4B39-A8D0-E1D42DE1D5F3`, propID 5, type String).[^87] The only Microsoft page showing the raw INI layout at all is the Mixed Reality 3D-app-launcher documentation, which prints a sample .url containing `[{9F4C2855-…}] Prop31=… Prop5=…` and `[{000214A0-…}] Prop3=19,0` without explaining the syntax.[^88]

## 4.2 VT type tags observed on disk

| Decimal tag | Hex | VARTYPE | Serialized form on disk | Where observed | Confidence |
|---|---|---|---|---|---|
| 1 | 0x0001 | VT_NULL | none — the fixture carries a dummy `,2` | Wine test fixture `Prop0=1,2` (artificial; parser ignores it)[^89] | [RE'd] |
| 3 | 0x0003 | VT_I4 | decimal integer | `{A7AF692E}` Prop5=3,0 / Prop6=3,1 in .website files[^90][^91] | [Observed] |
| 8 | 0x0008 | VT_BSTR | literal string | `{9F4C2855}` Prop5=8,Microsoft.Website.…[^87][^90] | [Observed] |
| 19 | 0x0013 | VT_UI4 | decimal integer | Prop3=19,* in FMTID_Intshcut; Prop9=19,* rating[^92][^93] | [Observed] |
| 31 | 0x001F | VT_LPWSTR | literal Unicode string | Prop4=31,<title>; Prop21=31,<desc>; SummaryInformation strings[^92][^94] | [Observed] |
| 65 | 0x0041 | VT_BLOB | lowercase hex bytes | `{A7AF692E}` Prop2=65,<hex> in .website files[^90][^91] | [Observed] |
| 4127 | 0x101F | VT_VECTOR\|VT_LPWSTR | semicolon-joined strings | `{F29F85E0}` Prop4/Prop5 (Author, Keywords)[^94][^95] | [Observed] |

The tag set is exactly the OLE `VARENUM` space (0x1000 = VT_VECTOR, 0x001F = VT_LPWSTR, so 4127 = 0x101F is a vector of Unicode strings — matching the multi-valued Author/Keywords properties).[^94][^95] The decimal-tag→VARTYPE mapping is inference from the OLE headers, anchored by two independent confirmations: the documented variant types of PID_IS_NAME (VT_LPWSTR ↔ 31) and System.AppUserModel.ID (String ↔ 8),[^85][^87] and the voidtools Everything documentation, which independently uses the same `Prop<N> = 31,<value>` syntax for the same GUIDs.[^95] Caveat: the tag does not discriminate semantics across sections — tag 19 carries an undocumented magic value in FMTID_Intshcut (Prop3) but a documented star rating in `{64440492-…}` (Prop9).[^92][^93]

## 4.3 FMTID_Intshcut [{000214A0-…}]: the documented PID_IS_* table

FMTID_Intshcut = `{000214A0-0000-0000-C000-000000000046}` is defined in the SDK header `ShlGuid.h`; the PROPID constants are in `ShlObj.h`; the variant types and descriptions are on the Microsoft Learn "Internet Shortcuts" page (now under the Legacy Windows Environment Features node).[^85][^86][^96]

| Constant | PROPID | VT (documented) | Description | Confidence |
|---|---|---|---|---|
| PID_IS_URL | 2 | VT_LPWSTR | URL to which the shortcut leads | [Documented] |
| — | 3 | — | **No documented name** (see [§4.5](#45-observed-but-undocumented-property-ids)) | [Documented] (absence) |
| PID_IS_NAME | 4 | VT_LPWSTR | Name of the Internet shortcut | [Documented] |
| PID_IS_WORKINGDIR | 5 | VT_LPWSTR | Working directory for the shortcut | [Documented] |
| PID_IS_HOTKEY | 6 | VT_UI2 | Hotkey for the shortcut | [Documented] |
| PID_IS_SHOWCMD | 7 | VT_I4 | Show command for shortcut | [Documented] |
| PID_IS_ICONINDEX | 8 | VT_I4 | Index of the icon | [Documented] |
| PID_IS_ICONFILE | 9 | VT_LPWSTR | File that contains the icon | [Documented] |
| PID_IS_WHATSNEW | 10 | VT_LPWSTR | What's New text | [Documented] |
| PID_IS_AUTHOR | 11 | VT_LPWSTR | Author | [Documented] |
| PID_IS_DESCRIPTION | 12 | VT_LPWSTR | Description text of site | [Documented] |
| PID_IS_COMMENT | 13 | VT_LPWSTR | User annotated comment | [Documented] |
| — | 14 | — | Skipped; no constant defined | [Documented] (absence) |
| PID_IS_ROAMED | 15 | VT_BOOL | True when shortcut is roamed for first time | [Documented] |

Two documentation anomalies are worth flagging. First, the Learn page lists names and types but not the numeric PROPIDs; the numbers come only from `ShlObj.h`, and the list skips 3 and 14 without comment.[^85][^86] Second, the `ShlObj.h` comment block states no variant type for PID_IS_ROAMED (its `#define` was appended later); the VT_BOOL typing comes only from the Learn page.[^85][^86] In practice, IE and Explorer almost never serialize this table's string properties: of the twelve documented PROPIDs, only Prop4 (PID_IS_NAME) is routinely seen on disk, while the most common property-bag line of all — Prop3 — is precisely the PROPID Microsoft never named, inverting the expectation that documented properties would dominate real files ([§4.5](#45-observed-but-undocumented-property-ids)).

## 4.4 FMTID_InternetSite [{000214A1-…}]: the documented PID_INTSITE_* table

FMTID_InternetSite = `{000214A1-0000-0000-C000-000000000046}` is defined alongside FMTID_Intshcut in `ShlGuid.h`.[^96] No real-world .url specimen in the research corpus contains a `[{000214A1-…}]` section; the table below is therefore API documentation, not an on-disk catalog.

| Constant | PROPID | VT (Learn page) | VT (header comment) | Notes | Confidence |
|---|---|---|---|---|---|
| PID_INTSITE_WHATSNEW | 2 | VT_LPWSTR | VT_LPWSTR | What's New text | [Documented] |
| PID_INTSITE_AUTHOR | 3 | VT_LPWSTR | VT_LPWSTR | Author | [Documented] |
| PID_INTSITE_LASTVISIT | 4 | VT_FILETIME | VT_FILETIME | Time site was last visited | [Documented] |
| PID_INTSITE_LASTMOD | 5 | VT_FILETIME | VT_FILETIME | Time site was last modified | [Documented] |
| PID_INTSITE_VISITCOUNT | 6 | VT_UI4 | VT_UI4 | Number of times user has visited | [Documented] |
| PID_INTSITE_DESCRIPTION | 7 | VT_LPWSTR | VT_LPWSTR | Description text of site | [Documented] |
| PID_INTSITE_COMMENT | 8 | VT_LPWSTR | VT_LPWSTR | User annotated comment | [Documented] |
| PID_INTSITE_FLAGS | 9 | VT_UI4 | (not in comment) | Carries PIDISF_ flags | [Documented] |
| PID_INTSITE_CONTENTLEN | 10 | N/A | (not in comment) | "Not currently supported" | [Documented] |
| PID_INTSITE_CONTENTCODE | 11 | N/A | (not in comment) | "Not currently supported" | [Documented] |
| PID_INTSITE_RECURSE | 12 | N/A | VT_UI4 | Levels to recurse (0–3); page says unsupported | [Documented] |
| PID_INTSITE_WATCH | 13 | N/A | VT_UI4 | PIDISM_ flags; page says unsupported | [Documented] |
| PID_INTSITE_SUBSCRIPTION | 14 | VT_UI8 | VT_UI8 | SUBSCRIPTIONCOOKIE for subscription manager | [Documented] |
| PID_INTSITE_URL | 15 | VT_LPWSTR | VT_LPWSTR | URL to which the shortcut leads | [Documented] |
| PID_INTSITE_TITLE | 16 | VT_LPWSTR | VT_LPWSTR | Title | [Documented] |
| — | 17 | — | — | Skipped | [Documented] (absence) |
| PID_INTSITE_CODEPAGE | 18 | VT_UI4 | VT_UI4 | Codepage of the document | [Documented] |
| PID_INTSITE_TRACKING | 19 | N/A | VT_UI4 | "Tracking"; page says unsupported | [Documented] |
| PID_INTSITE_ICONINDEX | 20 | VT_I4 | VT_I4 | Index of the icon | [Documented] |
| PID_INTSITE_ICONFILE | 21 | VT_LPWSTR | VT_LPWSTR | File that contains the icon | [Documented] |
| — | 22–33 | — | — | Skipped | [Documented] (absence) |
| PID_INTSITE_ROAMED | 34 | VT_UI4 | VT_UI4 | Entry was added due to roaming (PIDISR_ values) | [Documented] |

The header's comment block names one further property — `PID_INTSITE_RAWURL [VT_LPWSTR] "The raw, un-encoded, unicode url."` — but **no `#define PID_INTSITE_RAWURL` exists** in the 10.0.10240.0 SDK header, and the Learn page omits it entirely; the constant survives only as a comment, so its PROPID is unknowable from public sources.[^86] The page's "N/A / not currently supported" for RECURSE, WATCH and TRACKING contradicts the header comments (VT_UI4 with real semantics); read the page as describing implementation status, not type information.[^85][^86]

Associated flag constants from `ShlObj.h`:[^86]

| Constant family | Values | Applies to | Confidence |
|---|---|---|---|
| PIDISF_ (site flags) | RECENTLYCHANGED=0x1, CACHEDSTICKY=0x2, CACHEIMAGES=0x10, FOLLOWALLLINKS=0x20 | PID_INTSITE_FLAGS (9) | [Documented] |
| PIDISM_ (watch mode) | GLOBAL=0, WATCH=1, DONTWATCH=2 | PID_INTSITE_WATCH (13) | [Documented] |
| PIDISR_ (roaming state) | UP_TO_DATE=0, NEEDS_ADD=1, NEEDS_UPDATE=2, NEEDS_DELETE=3 | PID_INTSITE_ROAMED (34) | [Documented] |

The Learn page marks PIDISF_CACHEDSTICKY, PIDISF_CACHEIMAGES and PIDISF_FOLLOWALLLINKS as "not currently supported", and documents the PIDISR_ values as the roaming-history state machine (e.g. PIDISR_NEEDS_DELETE instructs the roamer to remove the entry via `DeleteUrlCacheEntry`).[^85]

## 4.5 Observed-but-undocumented property IDs

The properties that actually dominate real files are the ones Microsoft does not document.

**The Prop3 mystery (FMTID_Intshcut, PROPID 3).** PROPID 3 has no `PID_IS_*` constant — the SDK list jumps from 2 to 4 — yet `Prop3=19,<n>` (VT_UI4) is the most common property-bag line in existence, written by IE, Explorer, Steam, Epic Games Launcher and countless generators.[^84][^86][^92] All observed values:

| Value | Context / specimen | Confidence |
|---|---|---|
| `Prop3=19,0` | MS Mixed Reality launcher sample;[^88] Steam and Epic Games shortcuts[^97][^98] | [Observed] |
| `Prop3=19,2` | Typical IE-created favorite; MS KB sample; Check Point `xd.website`[^90][^93][^99] | [Observed] |
| `Prop3=19,9` | Quarkslab MS16-104 PoC; in-the-wild Blind Eagle / CVE-2024-43451 samples (in a *malformed* GUID section `{009862A0-…}`)[^99][^100] | [Observed] |
| `Prop3=19,11` | metabpa bookmark sample; multiple committed GitHub .url/.website files[^92][^101] | [Observed] |
| `Prop3=19,15` | IE bookmarklet .url files[^102] | [Observed] |

No source — official or community — explains Prop3's semantics. Quarkslab's binary analysis of the MS16-104 exploit assigns it no role: the PoC carries `Prop3=19,9` while a benign shortcut carries `Prop3=19,2`, and the bypass is entirely attributable to the un-prompted `ShellExecuteEx` of the `URL` target.[^99] Community forums explicitly admit the meaning is unknown ("I also don't know what the Prop3= key is for").[^103] The only defensible statement is: undocumented PROPID 3, VT_UI4, values 0/2/9/11/15 observed, semantics unknown.

**Prop4=31,<title> — the confirmed mapping.** `Prop4=31,Google`, `Prop4=31,go.microsoft.com`, `Prop4=31,Stack Overflow - Where Developers Learn…` appear in .website files and IE-specific .url files.[^90][^91][^104] PROPID 4 = PID_IS_NAME (VT_LPWSTR) is documented,[^85][^86] and the observed values are page titles — making Prop4 the anchor confirming the whole `Prop<N>` = PROPID N decoding.

**Extended shell property sets.** Three further GUID sections appear in Explorer-annotated favorites, documented by metabpa and corroborated by a Microsoft support KB (the "apply-property-error" article, near-certainly the one metabpa cites — same GUIDs, Prop numbers, star values, sample URL):[^92][^93]

| Section GUID | Line | Property | Confidence |
|---|---|---|---|
| {5CBF2787-48CF-4208-B90E-EE5E5D420294} | `Prop21=31,<text>` | System.Link.Description (Explorer "Description" field) | [Observed] |
| {B9B4B3FC-2B51-4A42-B5D8-324146AFCF25} | `Prop5=31,<text>` | System.Link.Comment ("Notes" field) | [Observed] |
| {64440492-4C8B-11D1-8B70-080036B11A03} | `Prop9=19,{1,25,50,75,99}` | System.Rating — 1 to 5 stars | [Observed] |

The star-rating scale (1/25/50/75/99 for 1–5 stars) is given identically by metabpa and the MS KB.[^92][^93] The property-set GUIDs are not defined in any located SDK header; the System.* names are community/metabpa attributions consistent with the Windows property-system naming scheme — treat the GUID→canonical-name binding as inferred, the GUID→field binding as observed.

**SummaryInformation.** A Microsoft Q&A specimen shows a `[{F29F85E0-4FF9-1068-AB91-08002B27B3D9}]` section (the standard FMTID_SummaryInformation) carrying `Prop2=31` Title, `Prop3=31` Subject, `Prop4=4127` Author, `Prop5=4127` Keywords, `Prop6=31` Comment — the classic OLE document-summary PROPIDs 2–6, with 4127 = VT_VECTOR|VT_LPWSTR for multi-valued Author/Keywords.[^94][^95]

**Shadow sections.** Any GUID section may be duplicated as `[{GUID}.A]` (value in the system ANSI code page) and `[{GUID}.W]` (modified UTF-7: ASCII verbatim, non-ASCII Base64 between `+` and `-`, with `+`→`+-` and `-`→`-+` escaping). The NSIS wiki guessed this ("CP_ACP stuff?" / "UTF-7 stuff?"); the MS Q&A specimen (`…Description説明` vs `…Description+iqxmDg-`) and the tatsu-syo reverse-engineering page confirm it.[^84][^94][^105]

**Ignored lines.** Wine's own ieframe test fixture proves the reimplemented parser discards the property-bag layer entirely: a fixture containing `[{000214A0-…}] Prop0=1,2` is loaded, after which PID_IS_ICONFILE/PID_IS_ICONINDEX reads return `S_FALSE`/`VT_EMPTY` — the GUID section is not deserialized.[^89]

## 4.6 The .website property sections

.website files (IE9+ pinned sites) reuse the .url grammar and add two property sets.[^90][^91][^106] Verbatim in-the-wild specimen (Check Point, `xd.website`, CVE-2025-24054 campaign):[^90]

```
[{000214A0-0000-0000-C000-000000000046}]
Prop3=19,2
Prop4=31,go.microsoft.com
[InternetShortcut]
URL=file://159.196.128[.]120/
[{A7AF692E-098D-4C08-A225-D433CA835ED0}]
Prop5=3,0
Prop9=19,0
Prop2=65,2C0000000000000001000000FFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFF310000002B000000710600006204000056
Prop6=3,1
[{9F4C2855-9F79-4B39-A8D0-E1D42DE1D5F3}]
Prop5=8,Microsoft.Website.B4BD2547.99055A5E
```

| Section / line | Decoding | Confidence |
|---|---|---|
| `{A7AF692E-098D-4C08-A225-D433CA835ED0}` | .website-only property set; **no Microsoft documentation found** (absent from ShlGuid.h, Propkey.h, and all located Learn pages) | [Observed] |
| `Prop5=3,0` / `Prop6=3,1` | PROPID 5/6, VT_I4; constants present in every full specimen; semantics unknown | [Observed] |
| `Prop9=19,0` | PROPID 9, VT_UI4, always 0 in specimens; semantics unknown | [Observed] |
| `Prop2=65,<hex>` | PROPID 2, VT_BLOB; 45 bytes in the Check Point specimen: LE uint32 length 0x2C (44), then 40 struct bytes (two leading LE uint32s 0 and 1, 0xFF×16 padding, four trailing LE uint32s), plus one trailing byte — layout not confirmed by any source | [Observed] |
| `{9F4C2855-…}` `Prop5=8,Microsoft.Website.<8hex>.<8hex>` | System.AppUserModel.ID (documented property key: formatID 9F4C2855…, propID 5, type String); the `Microsoft.Website.*` AUMID naming convention itself is undocumented | [Documented] (key) / [Observed] (naming) |
| `{9F4C2855-…}` `Prop31=<path>` + `Prop5=<AUMID>` (no type tag) | Mixed Reality launcher .url: VisualElementsManifest path + ExplicitAppUserModelID; note these lines **omit the `<vt>,` prefix** in Microsoft's sample | [Documented] (sample) |

The Prop2 blob's leading `2C000000` (= 44) accounts for its own 4 bytes plus the 40 struct bytes that follow, leaving one trailing byte (`56` in the Check Point specimen) — the same length-plus-trailing-byte pattern as the `IDList=`/`Modified=` hex encodings, which the unofficial documentation attributes to `WritePrivateProfileStruct` with a checksum byte. The variable tail (`…710600006204000056` vs `…25060000690300006A` in the Spiceworks google.com sample) suggests embedded counters or coordinates, but no source decodes the blob, so any structural reading is speculative.[^90][^106] Benign IE9-era specimens (e.g. the beckus WebShortcutSamples `Google.website`) show the same sections minus Prop2/Prop6 in some files, so only Prop5/Prop9 of `{A7AF692E}` appear mandatory.[^91][^107]

## 4.7 Confirmed vs inferred: mapping summary

| Mapping | Status |
|---|---|
| PropN = legacy PROPID N within the section's FMTID | Confirmed-by-specimen (Prop4=PID_IS_NAME, Prop5=System.AppUserModel.ID) |
| Decimal tag = OLE VARTYPE (3/8/19/31/65/4127) | Confirmed-by-specimen (matches documented VTs of PID_IS_NAME, System.AppUserModel.ID) |
| PID_IS_* table ([§4.3](#43-fmtid_intshcut-000214a0--the-documented-pid_is_-table)), PID_INTSITE_* table ([§4.4](#44-fmtid_internetsite-000214a1--the-documented-pid_intsite_-table)), PIDISF_/PIDISM_/PIDISR_ | Confirmed-documented (Learn + ShlObj.h) |
| FMTID_Intshcut / FMTID_InternetSite GUID values | Confirmed-documented (ShlGuid.h) |
| Prop4=31,<title> = PID_IS_NAME | Confirmed-documented + observed |
| Prop3=19,* (FMTID_Intshcut) semantics | Unknown (no documented name; values 0/2/9/11/15 observed; no exploit role per Quarkslab) |
| {5CBF2787} Prop21 = Description; {B9B4B3FC} Prop5 = Notes; {64440492} Prop9 = Rating | Confirmed-by-specimen (metabpa + MS KB); System.* canonical names inferred |
| {F29F85E0} Prop2–6 = Title/Subject/Author/Keywords/Comment | Confirmed-by-specimen (MS Q&A; standard SummaryInformation FMTID) |
| {A7AF692E} Prop2/5/6/9 (.website metadata) | Unknown (observed; GUID undocumented) |
| {9F4C2855} Prop5 = System.AppUserModel.ID | Confirmed-documented (property key); `Microsoft.Website.*` naming observed-only |
| Prop0=1,2 (Wine fixture) | Ignored by parser — [RE'd] negative result |
| PID_INTSITE_RAWURL | Inferred-to-be-vaporware: comment-only in ShlObj.h, no #define, absent from Learn |

---

## Footnotes

[^84]: https://nsis.sourceforge.io/Creating_internet_shortcuts — NSIS wiki, "Creating internet shortcuts" (section/key map; `Prop<2..2147483647>`; FMTID_Intshcut label)
[^85]: https://learn.microsoft.com/en-us/windows/win32/lwef/internet-shortcuts — Microsoft Learn, "Internet Shortcuts" (PID_IS_* / PID_INTSITE_* names, variant types, PIDISF_/PIDISR_ tables)
[^86]: https://github.com/tpn/winsdk-10/blob/master/Include/10.0.10240.0/um/ShlObj.h — Windows SDK 10.0.10240.0 ShlObj.h mirror (numeric PID_IS_*/PID_INTSITE_* #defines, PIDISF_/PIDISM_/PIDISR_ constants, PID_INTSITE_RAWURL comment)
[^87]: https://learn.microsoft.com/en-us/windows/win32/properties/props-system-appusermodel-id — Microsoft Learn, System.AppUserModel.ID (formatID 9F4C2855-9F79-4B39-A8D0-E1D42DE1D5F3, propID 5, type String)
[^88]: https://learn.microsoft.com/en-us/windows/mixed-reality/distribute/implementing-3d-app-launchers-win32 — Microsoft Learn, "Implementing 3D app launchers" (sample .URL launcher with Prop31/Prop5 in {9F4C2855} and Prop3=19,0)
[^89]: https://github.com/wine-mirror/wine/blob/master/dlls/ieframe/tests/intshcut.c — Wine ieframe test fixture (`Prop0=1,2` loaded then asserted ignored)
[^90]: https://research.checkpoint.com/2025/cve-2025-24054-ntlm-exploit-in-the-wild/ — Check Point Research, "CVE-2025-24054 NTLM Exploit in the Wild" (verbatim xd.website specimen)
[^91]: https://community.spiceworks.com/t/microsoft-edge-desktop-shortcut/790553 — Spiceworks community (verbatim google.com .website specimen)
[^92]: http://www.metabpa.org/projects/psbrowserbookmarks/about_browserbookmarks — metabpa.org browser bookmarks project (extended-property sections, star-rating scale, Prop3=19,11 sample)
[^93]: https://learn.microsoft.com/zh-cn/previous-versions/troubleshoot/browsers/core-features/apply-property-error — Microsoft Support KB "apply-property-error" (Description/Notes/Rating Prop mappings, Prop3=19,2 sample)
[^94]: https://learn.microsoft.com/ja-jp/answers/questions/4115480/question-4115480 — Microsoft Q&A (Japanese) full property-set dump incl. {F29F85E0} SummaryInformation and .A/.W shadow sections
[^95]: https://www.voidtools.com/support/everything/properties/ — voidtools Everything property documentation (Prop<N> = 31,<value> syntax for the same GUIDs)
[^96]: https://github.com/tpn/winsdk-10/blob/master/Include/10.0.10240.0/um/ShlGuid.h — Windows SDK 10.0.10240.0 ShlGuid.h mirror (FMTID_Intshcut / FMTID_InternetSite GUID definitions)
[^97]: https://gist.github.com/596dd5645906cef4716af4abf58df661 — Steam shortcut gist (Prop3=19,0)
[^98]: https://forum.kodi.tv/showthread.php?tid=287826&page=120 — Kodi forum paste of Epic Games Launcher .url (Prop3=19,0)
[^99]: https://blog.quarkslab.com/analysis-of-ms16-104-url-files-security-feature-bypass-cve-2016-3353.html — Quarkslab, "Analysis of MS16-104" (Prop3=19,9 PoC vs Prop3=19,2 benign; no exploit role for Prop3)
[^100]: https://research.checkpoint.com/2025/blind-eagle-and-justice-for-all/ — Check Point Research, Blind Eagle campaign (Prop3=19,9 in malformed {009862A0-…} section, in the wild)
[^101]: https://github.com/kanalstrahlen/precisION/blob/main/Tutorial2.url — committed .url specimen (Prop3=19,11); also https://github.com/jx-admin/Code2/blob/master/androidCode.url
[^102]: https://gist.github.com/dungsaga/45788260c81832de54ab5e237b523d22 — IE bookmarklet .url gist (Prop3=19,15; ExtendedURL)
[^103]: https://arstechnica.com/civis/threads/whats-the-trick-for-creating-large-icons-for-xp.185275/ — Ars Technica forum ("I also don't know what the Prop3= key is for")
[^104]: https://stackoverflow.com/questions/62490091 — Stack Overflow specimen (Explorer-written file, Prop4=31,<page title>, section order 000214A0 → A7AF692E → InternetShortcut → 9F4C2855)
[^105]: https://www.tatsu-syo.info/Devroom/IEfavorites.html — tatsu-syo.info IE Favorites reverse engineering (.W section modified-UTF-7 encoding)
[^106]: https://github.com/JONGGON/DeepHumanPrediction — committed IE9-era .website specimen (full {A7AF692E} section with Prop2 blob); also https://github.com/codejanus/study_Security (.website with Prop6=3,1)
[^107]: https://github.com/beckus/WebShortcutSamples — WebShortcutSamples repo (hex-verified IE9 Google.website; README: format undocumented)

<!-- chapter-nav -->

---

← [3. Keys](03-keys.md) · [Chapters](README.md) · [5. Security behaviors →](05-security-behaviors.md)
