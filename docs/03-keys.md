# 3. Keys

> Part of the [catalog](../README.md). [Chapters](README.md) · [Bibliography](../references.md) · [Specimens](../specimens/README.md)
>
> ← [2. Sections](02-sections.md) · **3. Keys** · [4. Property bags →](04-property-bags.md)

## Contents

- [3.1 Master keys table](#31-master-keys-table)
- [3.2 URL and the length limits](#32-url-and-the-length-limits)
- [3.3 The Modified field: inverted FILETIME + checksum byte](#33-the-modified-field-inverted-filetime--checksum-byte)
- [3.4 The HotKey encoding](#34-the-hotkey-encoding)
- [3.5 Icon fields (IconFile, IconIndex)](#35-icon-fields-iconfile-iconindex)
- [3.6 Execution-context fields (WorkingDirectory, ShowCommand)](#36-execution-context-fields-workingdirectory-showcommand)
- [3.7 Metadata fields (IDList, Roamed, Author, WhatsNew, Comment, Desc)](#37-metadata-fields-idlist-roamed-author-whatsnew-comment-desc)
- [3.8 Frameset and feature keys (BASEURL, ORIGURL, ExtendedURL, FeedUrl, IsLivePreview, PreviewSize)](#38-frameset-and-feature-keys-baseurl-origurl-extendedurl-feedurl-islivepreview-previewsize)


This chapter catalogs every `name=value` key observed in `.url` and `.website` files: which section owns it, the value's type and format, its exact semantics, the behavior when it is absent, which software writes it, and any length limits. Confidence labels follow the catalog convention: **[Documented]** (Microsoft documentation or SDK headers), **[Observed]** (seen in real files), **[RE'd]** (established by binary or source analysis of Wine/ReactOS), **[Rumor]** (claimed but unverified). Because the format has no published specification, the documented anchor for most keys is the `FMTID_Intshcut` property set: Microsoft Learn and the Windows SDK header `ShlObj.h` define `PID_IS_URL`, `PID_IS_NAME`, `PID_IS_WORKINGDIR`, `PID_IS_HOTKEY`, `PID_IS_SHOWCMD`, `PID_IS_ICONINDEX`, `PID_IS_ICONFILE`, `PID_IS_WHATSNEW`, `PID_IS_AUTHOR`, `PID_IS_DESCRIPTION`, `PID_IS_COMMENT`, and `PID_IS_ROAMED` with their variant types, and the INI keys of `[InternetShortcut]` are the serialized form of those same properties.[^52][^53]

## 3.1 Master keys table

| Key | Owning section | Value type / format | Semantics | Default when absent | Written by | Length limit | Confidence |
|---|---|---|---|---|---|---|---|
| `URL` | `[InternetShortcut]` | String (ANSI/UTF-8/UTF-16 file encodings all observed) | Target address the shortcut opens; scheme-agnostic | None — the only required key; load fails without it | Everything: Explorer drag-from-address-bar, IE favorites, Edge, Steam, Epic, installers, malware | 2083 chars [Observed] | [Documented] (PID_IS_URL, VT_LPWSTR)[^52] |
| `IconFile` | `[InternetShortcut]` | String path or URL | File containing the shortcut's icon (ICO/DLL/EXE); local, UNC, and http(s) favicon URLs all observed | `%SystemRoot%\system32\url.dll,0` (registry DefaultIcon fallback) | IE/Edge favorites (favicon URL), Steam/Epic (exe path), attackers (UNC) | None found | [Documented] (PID_IS_ICONFILE)[^52][^54] |
| `IconIndex` | `[InternetShortcut]` | Decimal integer | Zero-based index of the icon within `IconFile`; parsed with `wcstol`, junk → 0 | 0 | Same as IconFile | None found | [Documented] (PID_IS_ICONINDEX, VT_I4)[^52][^55] |
| `HotKey` | `[InternetShortcut]` | Decimal integer: `(HOTKEYF_flags << 8) \| VK` | Keyboard shortcut that launches the .url | 0 / absent = no hotkey | IE favorites property sheet; `HotKey=0` written routinely by IE | 16-bit value space | [Observed] (encoding formula per [§3.4](#34-the-hotkey-encoding))[^56][^57] |
| `ShowCommand` | `[InternetShortcut]` | Decimal integer | Window show state: 3 = maximized, 7 = minimized | Absent = normal window | IE/Windows shortcut property sheet (absent in some IE/Windows versions) | — | [Observed]; 1 = SW_NORMAL disputed ([§3.6](#36-execution-context-fields-workingdirectory-showcommand))[^56][^58] |
| `WorkingDirectory` | `[InternetShortcut]` | String path | Working folder for the launching application; IE ignores it | None | Older IE/Windows, Epic Games Launcher, ExifTool test fixture | None found | [Documented] (PID_IS_WORKINGDIR)[^52][^56] |
| `Modified` | `[InternetShortcut]` | 18 hex chars = 8 bytes byte-reversed FILETIME + 1 checksum byte | Last-modified timestamp of the shortcut data | Absent; not rewritten by modern Windows | IE4–IE6-era favorites; rare in modern files | Fixed 18 hex chars | [Observed] (decode per [§3.3](#33-the-modified-field-inverted-filetime--checksum-byte))[^56][^59] |
| `IDList` | `[InternetShortcut]` | Hex string (WritePrivateProfileStruct-style + checksum byte) | Shell item ID list; Explorer prefers it to locate the resource; almost always empty on disk | Empty | Explorer (drag-created local `file://` shortcuts) | — | [Observed][^60][^61] |
| `Roamed` | `[InternetShortcut]` | Integer/bool (`-1`, `1` observed) | "True when shortcut is roamed for first time" (roaming-profiles legacy) | Absent = not roamed | IE favorites (roaming era) | — | [Documented] (PID_IS_ROAMED, VT_BOOL)[^52][^62] |
| `Author` | `[InternetShortcut]` | String | Author metadata | Absent | IE favorites / Active Desktop legacy; no longer parsed per community survey | — | [Documented] (PID_IS_AUTHOR)[^52][^61] |
| `WhatsNew` | `[InternetShortcut]` | String | "What's New" text for the site | Absent | IE favorites legacy | — | [Documented] (PID_IS_WHATSNEW)[^52] |
| `Comment` | `[InternetShortcut]` | String | User-annotated comment; old property-sheet tooltip, not shown on Win10/11 | Absent | IE favorites property sheet (legacy) | — | [Documented] (PID_IS_COMMENT)[^52][^61] |
| `Desc` | `[InternetShortcut]` | String | Description text of site | Absent | IE favorites legacy | — | [Documented] (PID_IS_DESCRIPTION)[^52] |
| `BASEURL` | `[DEFAULT]`, `[DOC#n(#n…)]` | String URL | Frameset state: absolute URL of the page/frame document | Absent (non-framed pages) | IE favorites, when saving framed pages | — | [Observed][^56][^63] |
| `ORIGURL` | `[DOC#n(#n…)]` | String (often relative) | Original (possibly relative) URL of the frame document | Absent | IE favorites, framed pages | — | [Observed][^56] |
| `ExtendedURL` | `[Bookmarklet]` | String (javascript:) | Full bookmarklet script when it exceeds the `URL` limit | Absent (non-bookmarklet) | IE11 favorites (bookmarklets) | 5119 bytes (IE11) [Observed] | [Observed][^54][^64] |
| `FeedUrl` | `[MonitoredItem]` | String URL | Feed/Web Slice source URL (note lowercase "rl" spelling) | Absent | IE8+ Web Slices / feed monitor | — | [Observed][^54][^65] |
| `IsLivePreview` | `[MonitoredItem]` | `true`/`false` | Marks the favorite as a live-preview Web Slice | Absent | IE8+ Web Slices | — | [Observed][^65][^66] |
| `PreviewSize` | `[MonitoredItem]` | `<w>x<h>` (e.g. `320x240`) | Web Slice preview dimensions | Absent | IE8+ Web Slices | — | [Observed][^66] |

Two read/write asymmetries apply to the whole table. First, the only keys the Wine and ReactOS reimplementations of `CInternetShortcut` actually *read* are `URL`, `iconfile`, and `iconindex`; every other key in this table — `Modified`, `HotKey`, `ShowCommand`, `WorkingDirectory`, `IDList`, `Roamed`, the metadata strings, and all `Prop<N>=` property-storage lines — is ignored by both codebases, a negative result established by tree-wide greps and by a Wine test fixture that loads a file containing `HotKey=0`, `IDList=`, and a `[{000214A0-…}] Prop0=1,2` section and then asserts the properties come back empty.[^55][^67] Second, on *write* both reimplementations emit the icon keys in uppercase (`ICONFILE=`, `ICONINDEX=%d`) while reading them back in lowercase — a round-trip that only works because `GetPrivateProfileStringW` folds case.[^55] Real Microsoft `ieframe.dll` is the only implementation known to honor the full key set.

Several conflicts are visible in the table. `ShowCommand=1` is listed by vpscoder but not by Blake ([§3.6](#36-execution-context-fields-workingdirectory-showcommand)). `Roamed` appears as `-1` in metabpa's reference file but as `1` in the ExifTool test fixture, while the documented type is VT_BOOL — the on-disk integer serialization of the boolean is not documented. `Author`/`WhatsNew`/`Comment`/`Desc` are documented as property IDs yet a community field-status survey reports they are no longer parsed on modern Windows, and ctrl.blog notes `Desc=` "isn't displayed or used for anything in any modern operating system."[^61][^68] Keys claimed by the format-aggregator site filetypedb (`Referer=`, `BrowserFlags=`, `BaseURL=` inside `[InternetShortcut]`) are contradicted by every specimen and by the absence of matching PROPIDs; they are quarantined in the rumor table ([chapter 6, §6.1](07-rumors-versions-gaps.md#61-unverified-and-rumored-keys)).[^69]

## 3.2 URL and the length limits

`URL` is the only mandatory key: the NSIS wiki states "Only the URL value in the InternetShortcut section is required, everything else is optional," and the Wine/ReactOS load path fails outright if `URL` is missing or unreadable.[^54][^55] The value is scheme-agnostic. Blake's guide already noted that "a URL file is not restricted to the HTTP protocol,"[^56] and specimens confirm `http(s):`, `file:` (including the malicious `URL=file://159.196.128[.]120/` of the Check Point .website specimen), `steam://rungameid/730`, `com.epicgames.launcher://apps/Fortnite?action=launch&silent=true`, `javascript:` bookmarklets, and `MyLauncher://launch/app-identifier` from Microsoft's own Mixed Reality documentation.[^70][^71][^72] The rclip-url-file parser documentation adds that bare paths occur — "URL=file:///C:/x and even a bare URL=C:\x both occur" — so consumers must not assume a scheme is present.[^60]

The operative length limit is **2083 characters** for `URL`, versus **5119** for `[Bookmarklet] ExtendedURL`. The NSIS wiki states: "Bookmarklet:ExtendedURL allows bookmarklets up to 5119 characters while the normal InternetShortcut:URL field is limited to 2083 characters."[^54] A real bookmarklet specimen gist refines the unit: "IE11 in Windows 10 allows favorite to contain URL upto 5119 **bytes** (not characters). It's stored in ExtendedURL. URL contains a truncated string (maximum 2083 bytes) of ExtendedURL. The tooltip is further truncated (down to 255 bytes). If you change ExtendedURL, but forget to update URL, then the bookmarklet won't work anymore."[^64] A bookmarklet generator blog corroborates the pair and adds that files containing CJK script must be saved as UTF-16LE, not UTF-8.[^73] By contrast, the Wine/ReactOS parser imposes no fixed limit at all — its `get_profile_string` helper doubles a 128-WCHAR buffer until the value fits — so the 2083 figure is a behavior of Microsoft's `ieframe.dll`/IE favorites UI, not of the format itself.[^55] The 2083 limit is labeled [Observed]: it originates from community measurement (NSIS wiki, gist) and matches the well-known `INTERNET_MAX_URL_LENGTH` constant family, but no Microsoft page states it for .url files specifically.

## 3.3 The Modified field: inverted FILETIME + checksum byte

`Modified` is an 18-character hex string encoding nine bytes: the first eight are a **FILETIME** (a 64-bit count of 100-nanosecond intervals since 1601-01-01 UTC) stored in **byte-reversed** order, and the ninth is a checksum byte that Blake's guide calls "unimportant."[^56] Decode algorithm:

1. Split the hex string into 9 bytes; drop the 9th (checksum).
2. Reverse the order of the first 8 bytes.
3. Interpret the result as a little-endian FILETIME and pass it to `FileTimeToSystemTime`.

Worked example from Blake's 3rd edition, independently re-verified during research:[^56]

```
Raw:      C0 34 90 B3 07 DC C3 01 | DE
Inverted: 01 C3 DC 07 B3 90 34 C0        (DE removed)
FILETIME: 0x01C3DC07B39034C0 = 127187140131960000
→ 2004-01-16 08:06:53 UTC
```

The guide's decode table prints the low dword as "B3 90 34 C30" — a typo in the source (trailing `C30` for `C0`); the arithmetic only closes with `C0`. A second example value in the same guide's running text, `Modified=20F06BA06D07BD014D`, decodes to FILETIME 125264532060500000 → 1997-12-13 02:20:06 UTC with checksum byte `0x4D`; the guide uses two different example values (text vs. table), which is a quirk of the source, not an extraction error.[^56] The 2nd edition of the guide (lyberty mirror) preserves the pre-FILETIME analysis — a guessed "poor man's" formula with a fitted constant `k = 10.2215` — and quotes Mat Kramer's 29 Aug 2000 email first proposing the FILETIME explanation, which the 3rd edition adopted.[^74]

The rclip-url-file crate documents the practitioner interpretation: the value is "a hex-encoded FILETIME plus a trailing byte. The bytes are stored in little-endian order, so the hex string reads backwards… The unofficial documentation calls the ninth byte a checksum; nothing here depends on it, so it is handed back raw rather than validated."[^60] Neither Wine nor ReactOS contains any `Modified=` parsing, FILETIME inversion, or checksum code at all.[^55] `Modified` is essentially absent from modern Explorer/Edge-written files; it is a hallmark of IE4–IE6-era favorites and of files written by tools imitating that era (e.g. the ExifTool test fixture `Modified=20128462C7D2BE0170`).[^62]

## 3.4 The HotKey encoding

`HotKey` is a decimal integer packing a virtual-key code in the low byte and modifier flags in the high byte:

```
HotKey = (HOTKEYF_flags << 8) | VK
```

where the modifier bits match the Win32 `HOTKEYF_*` constants (Shift = 0x01, Ctrl = 0x02, Alt = 0x04), giving modifier bases None = 0x000, Shift = 0x100 (256), Ctrl = 0x200 (512), Ctrl+Shift = 0x300 (768), Alt = 0x400 (1024), Shift+Alt = 0x500 (1280), Ctrl+Alt = 0x600 (1536), Ctrl+Shift+Alt = 0x700 (1792).[^56] The canonical examples: **833** = 0x341 = Ctrl+Shift+A, and **1601** = 0x641 = Ctrl+Alt+A. Blake's Appendix A tabulates all combinations for A–Z, 0–9, punctuation, and F1–F12; the formula above reproduces every published value except 30 entries in the digit block, which are systematically off by one (flagged below), and the brokenevent blog independently states the same encoding as a bitwise OR of `System.Windows.Forms.Keys` with Shift = 1<<8, Ctrl = 1<<9, Alt = 1<<10.[^56][^57] The rclip crate notes this is "the same encoding as ShellLinkHeader.HotKey in MS-SHLLINK" and that "files commonly carry HotKey=0."[^60]

One anomaly must be flagged. In the guide's digit block, the C+S, S+A and C+S+A columns are shifted by one for every digit 0–9: the published runs 817–826, 1329–1338 and 1841–1850 have low bytes 0x31–0x3A where the correct digit VKs are 0x30–0x39 — each digit row carries the next key's code, so the table prints Ctrl+Shift+0 as 817 where the formula gives 816. Only the C+A column (1584–1593, low bytes 0x30–0x39) is correct. The shift is present identically in both the 2nd and 3rd editions; treat the formula, not the table, as authoritative for digits (e.g. 1584 is the correct Ctrl+Alt+0 encoding).[^56][^74]

Who writes it: the hotkey is set through the IE favorites/shortcut property sheet, and IE-written files routinely carry `HotKey=0` (no hotkey assigned) — the rclip crate notes "files commonly carry HotKey=0," and the value appears in benign specimens and malware alike (e.g. the CVE-2024-43451 PoC).[^60][^75] Wine and ReactOS never read or write the key.[^55]

## 3.5 Icon fields (IconFile, IconIndex)

`IconFile` names the icon container and `IconIndex` selects within it. Blake documents the container as "generally … either a ICO, DLL or EXE file," with indexing zero-based and the default library being `URL.DLL` in the Windows\System directory.[^56] Three value classes are confirmed by specimens:

- **Local paths**: `IconFile=C:\Windows\System32\SHELL32.dll` (CVE-2024-43451 PoC), `IconFile=D:\Epic Games\Fortnite\FortniteGame\Binaries\Win64\FortniteLauncher.exe` (Epic launcher shortcut — an exe as icon source), Steam's `C:\Program Files (x86)\Steam\steam\games\<hash>.ico`.[^75][^71][^72]
- **UNC paths**: `IconFile=\\attacker\share\x.ico` — the NTLM-credential-leak vector; parsing happens at folder-enumeration time, not just on open (treated fully in the security chapter).[^76]
- **http(s) favicon URLs**: routine in IE/Edge favorites — `IconFile=https://github.global.ssl.fastly.net/favicon.ico`, `IconFile=https://cdn.sstatic.net/Sites/stackoverflow/img/favicon.ico?v=4f32ecc8f43d`, `IconFile=https://www.youtube.com/s/desktop/4468d336/img/favicon_32x32.png` (PNG, not just ICO).[^77][^78][^83] The rumor sweep verdict: **confirmed**.[^69]

`IconIndex` parsing is established by Wine/ReactOS source: the string is converted with `wcstol(iconindexstring, NULL, 10)`, so non-numeric junk silently becomes 0 [RE'd].[^55] Negative values — claimed by a cnblogs article (`IconIndex=-48`, "negative index means resource ID, not ordinal") by analogy with desktop.ini semantics — have **no captured specimen**; the rumor sweep rates them unverified/rumor-leaning, and they are carried in the rumor table rather than here.[^61][^69] When `IconFile` is absent, the shell falls back to the registry `DefaultIcon` (`%SystemRoot%\system32\url.dll,0`); brokenevent notes `IconIndex` is "meaningless without the IconFile."[^57][^79]

## 3.6 Execution-context fields (WorkingDirectory, ShowCommand)

`WorkingDirectory` is the working folder for the shortcut — "possibly the folder to be set as the current folder for the application that would open the file. However Internet Explorer does not seem to be affected by this field," per Blake, who also notes it "does not seem to appear in some versions of Internet Explorer/Windows."[^56] Real writers are third parties: Epic Games Launcher shortcuts carry `WorkingDirectory=D:\Epic Games` (and `C:\Program Files (x86)\Epic Games` in corroborating specimens), and the ExifTool test fixture carries `WorkingDirectory=C:\Users\Public\Documents`.[^71][^62] The key is documented as `PID_IS_WORKINGDIR` (VT_LPWSTR).[^52] This field is security-critical — it anchored the CVE-2025-33053 WebDAV hijack chain — and receives full treatment in [chapter 5](05-security-behaviors.md).

`ShowCommand` controls the window show state. The two sources conflict on one value and both are presented with attribution:

| Value | Blake (cyanwerks, 3rd ed.) | vpscoder (fileformat.vpscoder.com) |
|---|---|---|
| absent | Normal | — |
| 1 | *(not listed)* | Normal (SW_NORMAL) |
| 3 | Maximized | Maximized |
| 7 | Minimized | Minimized |

Blake's table lists only "(Nothing) Normal / 7 Minimized / 3 Maximized" and notes the setting "does not seem to appear in some versions of Internet Explorer/Windows."[^56] vpscoder lists "1 for normal, 3 for maximized, 7 for minimized."[^58] The value 1 = SW_NORMAL is plausible (it is the Win32 constant) but unverified in any captured specimen; the ExifTool fixture uses `ShowCommand=3`.[^62] The rclip crate keeps the raw number rather than an enum precisely because "the SW_* space is larger than the three values the unofficial documentation lists," and identifies 7 as SW_SHOWMINNOACTIVE — "what .url and .lnk both write for 'minimized'."[^60] `ShowCommand` is documented as `PID_IS_SHOWCMD` (VT_I4).[^52]

## 3.7 Metadata fields (IDList, Roamed, Author, WhatsNew, Comment, Desc)

These six keys are the INI serialization of documented `FMTID_Intshcut` properties — `PID_IS_WHATSNEW` (10), `PID_IS_AUTHOR` (11), `PID_IS_DESCRIPTION` (12), `PID_IS_COMMENT` (13), `PID_IS_ROAMED` (15) — plus `IDList`, which has no PROPID.[^52][^53] All six appear in the NSIS wiki's key list, and all six appear populated in a single specimen, the ExifTool test-suite file `LNK.url`:[^54][^62]

```
[InternetShortcut]
URL=https://www.example.com
IconFile=C:\Windows\System32\shell32.dll
IconIndex=15
WorkingDirectory=C:\Users\Public\Documents
HotKey=1582
ShowCommand=3
Modified=20128462C7D2BE0170
Author=Jane Doe
WhatsNew=Updated link and added new features to the site.
Comment=This is an example URL shortcut for demonstration purposes.
Desc=The official example website.
Roamed=1
IDList=2000000000000000
```

Notes per key:

- **IDList** — "Almost always empty in files on disk. The encoding is WritePrivateProfileStruct hex plus a checksum byte" (rclip).[^60] A Chinese-language field survey reports Explorer prefers IDList to locate the resource and that it appears often in drag-created local `file://` shortcuts.[^61] Most real files carry `IDList=` empty.[^77]
- **Roamed** — documented as VT_BOOL, "True when shortcut is roamed for first time."[^52] Observed serializations disagree: `Roamed=-1` (metabpa reference file) vs `Roamed=1` (ExifTool fixture).[^80][^62] A roaming-era IE favorites relic.
- **Author / WhatsNew / Comment / Desc** — documented as VT_LPWSTR properties,[^52] but the cnblogs survey reports `Comment`/`Desc` were the old property-sheet tooltip and are not shown on Windows 10/11, and `Author`/`WhatsNew`/`Roamed` are IE-favorites/Active-Desktop legacy no longer parsed.[^61] ctrl.blog similarly calls `Desc=` unused by any modern OS.[^68] Note that modern Explorer surfaces description/comment data through the GUID property-storage sections (`Prop21` in `{5CBF2787-…}`, `Prop5` in `{B9B4B3FC-…}`), not through these plain keys.[^80][^81]

## 3.8 Frameset and feature keys (BASEURL, ORIGURL, ExtendedURL, FeedUrl, IsLivePreview, PreviewSize)

**BASEURL / ORIGURL** preserve frameset state. When IE saved a favorite of a framed page, it added a `[DEFAULT]` section with `BASEURL=` and one `[DOC#n(#n#n…)]` section per frame with `BASEURL` (absolute) and `ORIGURL` (as-navigated, often relative); nested frames append number pairs (`[DOC#4#5#4#6]`).[^56] Blake's sample:[^56]

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

"The purpose of these extra fields is probably for the browser to figure out what HTML documents were loaded in each frame, since the main URL tends to not record the state of its framesets."[^56] A real IE8-generated favorite confirms `[DEFAULT] BASEURL=` in the wild.[^63] Placement matters: `BASEURL` inside `[InternetShortcut]` — as implied by filetypedb — is contradicted by every specimen and is filed as a rumor.[^69]

**ExtendedURL** (`[Bookmarklet]` section) holds the full `javascript:` bookmarklet when it exceeds the `URL` limit: 5119 bytes versus 2083 for `URL`, with `URL` carrying a truncated copy that must be kept in sync or "the bookmarklet won't work anymore" ([§3.2](#32-url-and-the-length-limits)).[^54][^64] Observed specimens also carry `Prop3=19,15` in the FMTID_Intshcut section — a Prop3 value otherwise unseen (0/2/9/11 elsewhere), apparently correlated with bookmarklets.[^64][^73]

**FeedUrl / IsLivePreview / PreviewSize** (`[MonitoredItem]` section) mark IE8-era Web Slice / feed-monitor favorites. Real Microsoft-shipped specimens recovered in sandbox analyses carry `FeedUrl=https://ieonline.microsoft.com/#ieslice`, and the "Suggested Sites.url" default link carries the full triple:[^65][^66]

```
[MonitoredItem]
FeedUrl=https://ieonline.microsoft.com/#ieslice
PreviewSize=320x240
IsLivePreview=true
```

The `FeedUrl` spelling (lowercase "rl") is consistent across all specimens, `IsLivePreview` takes literal `true`/`false`, and `PreviewSize` takes `<width>x<height>` (`320x240` observed). A minimal Web Slice containing only `FeedUrl` + `IsLivePreview=true` is also attested (SAP community specimen, `FeedUrl=http://scn.sap.com/`).[^82] `ieframe.dll` itself contains the template strings `FEEDURL="%s"`, `FeedViewer`, and `MonitoredItem`, confirming the shell/IE writes these sections.[^65]

---

## Footnotes

[^52]: https://learn.microsoft.com/en-us/windows/win32/lwef/internet-shortcuts — Microsoft Learn, "Internet Shortcuts" (Legacy Windows Environment Features): FMTID_Intshcut PID_IS_* property table with variant types.
[^53]: https://github.com/tpn/winsdk-10/blob/master/Include/10.0.10240.0/um/ShlObj.h — Windows 10 SDK 10.0.10240.0 ShlObj.h: numeric PID_IS_* (2,4–13,15) and PID_INTSITE_* definitions.
[^54]: https://nsis.sourceforge.io/Creating_internet_shortcuts — NSIS wiki, "Creating internet shortcuts": full section/key map; "Only the URL value … is required"; "Bookmarklet:ExtendedURL allows bookmarklets up to 5119 characters while the normal InternetShortcut:URL field is limited to 2083 characters."
[^55]: https://github.com/wine-mirror/wine/blob/master/dlls/ieframe/intshcut.c — Wine ieframe CInternetShortcut: reads only URL/iconfile/iconindex (`wcstol` for iconindex), writes URL=/ICONFILE=/ICONINDEX= uppercase, dynamic buffer (no URL length limit), no Modified/HotKey/ShowCommand code.
[^56]: http://www.cyanwerks.com/formats/file-format-url.html — Edward L. Blake, "An Unofficial Guide to the URL File Format," 3rd ed.: URL/WorkingDirectory/IconIndex/IconFile semantics, Modified FILETIME decode + worked example (C0 34 90 B3 07 DC C3 01 | DE → 16/1/2004 08:06:53, with the "C30" typo), ShowCommand table, HotKey Appendix A, frameset [DEFAULT]/[DOC#n] format.
[^57]: https://brokenevent.com/blog/2018-08-26 — The Broken Event Blog: HotKey as bitwise OR of System.Windows.Forms.Keys with Shift=1<<8, Ctrl=1<<9, Alt=1<<10; IconIndex "meaningless without the IconFile."
[^58]: https://fileformat.vpscoder.com/task-753 — fileformat.vpscoder.com, ".URL File Format": ShowCommand "1 for normal, 3 for maximized, 7 for minimized"; Modified as "inverted FILETIME structure format."
[^59]: https://docs.rs/rclip-url-file/latest/rclip_url_file/ — rclip-url-file crate docs: Modified as hex FILETIME + trailing byte handed back raw; ShowCommand kept as raw number, 7 = SW_SHOWMINNOACTIVE; HotKey = MS-SHLLINK encoding, "files commonly carry HotKey=0"; IDList encoding note; bare-path URL values; case-insensitivity and GetPrivateProfileString semantics.
[^60]: https://docs.rs/rclip-url-file/latest/rclip_url_file/ — rclip-url-file crate docs (fields module): IDList "Almost always empty … WritePrivateProfileStruct hex plus a checksum byte."
[^61]: https://www.cnblogs.com/suv789/p/18324691 — cnblogs field-status survey: Explorer prefers IDList; Comment/Desc not shown on Win10/11; Author/WhatsNew/Roamed legacy; negative IconIndex claim (unverified).
[^62]: https://raw.githubusercontent.com/exiftool/exiftool/master/t/images/LNK.url — ExifTool test-suite .url specimen with all metadata keys populated (Author/WhatsNew/Comment/Desc/Roamed=1/IDList/Modified/ShowCommand=3/HotKey=1582).
[^63]: https://ubuntugenius.wordpress.com/2009/12/09/how-to-open-url-internet-explorer-shortcuts-in-ubuntu-using-firefox/ — Real IE8-generated favorite with `[DEFAULT] BASEURL=http://www.google.com.au/`.
[^64]: https://gist.github.com/dungsaga/45788260c81832de54ab5e237b523d22 — Bookmarklet .url specimen gist: ExtendedURL/URL specimen with Prop3=19,15; "5119 bytes (not characters) … URL contains a truncated string (maximum 2083 bytes) … tooltip … 255 bytes."
[^65]: https://hybrid-analysis.com/sample/db1696106bb100a1fc10fadc9b93e17f80055604fa53fe3785b64efdabe0a254/56af7da60e316d585ed41a73 — Hybrid Analysis sandbox: Microsoft-shipped Web Slice .url files with `[MonitoredItem] FeedUrl=https://ieonline.microsoft.com/#ieslice`; ieframe.dll template strings FEEDURL/FeedViewer/MonitoredItem.
[^66]: http://forum.hotfix.pl/problemy/problem-ze-skrotem-uslugi-sugerowane-witryny-w-ie-t21332.html — Polish support forum reproducing the real "Suggested Sites.url": FeedUrl + PreviewSize=320x240 + IsLivePreview=true.
[^67]: https://github.com/wine-mirror/wine/blob/master/dlls/ieframe/tests/intshcut.c — Wine test fixture proving HotKey/IDList/GUID Prop sections are ignored on load.
[^68]: https://ctrl.blog/entry/internet-shortcut-files.html — ctrl.blog, "What is the best file format for web shortcuts": Title=/Desc= "aren't displayed or used for anything in any modern operating system."
[^69]: https://filetypedb.com/web/url — filetypedb.com "URL File Format": uncorroborated Referer=/BrowserFlags=/BaseURL-in-[InternetShortcut] claims (rumor-table entries); negative-IconIndex verdict per dim10 sweep.
[^70]: https://research.checkpoint.com/2025/cve-2025-24054-ntlm-exploit-in-the-wild/ — Check Point Research: in-the-wild malicious .website with `URL=file://159.196.128[.]120/`.
[^71]: https://forum.kodi.tv/showthread.php?tid=287826&page=120 — Kodi forum: Epic Games Launcher Fortnite.url specimen (com.epicgames.launcher:// URL, WorkingDirectory, exe IconFile, Prop3=19,0).
[^72]: https://landenlabs.com/cs-urlcleaner/urlcleaner.html — Steam .url specimens (`URL=steam://rungameid/<id>`, games\<hash>.ico IconFile).
[^73]: https://blog.darkthread.net/blog/ie-bookmarklet/ — darkthread blog: IE bookmarklet generator; 2083/5119 limits; UTF-16LE requirement for CJK.
[^74]: https://web.archive.org/web/20151224053029/http://www.lyberty.com/encyc/articles/tech/dot_url_format_-_an_unofficial_guide.html — Blake, 2nd ed. (lyberty mirror via Wayback): pre-FILETIME Modified analysis, Mat Kramer email (29 Aug 2000), same digit-block HotKey anomaly.
[^75]: https://gist.github.com/milo2012/00856e9273ab08829dc715a845abb4ed — CVE-2024-43451 PoC .url gist: HotKey=0, IconFile=C:\Windows\System32\SHELL32.dll, .A/.W section trick.
[^76]: https://swisskyrepo.github.io/InternalAllTheThings/active-directory/internal-shares/ — InternalAllTheThings: IconFile=\\attacker\share NTLM coercion on folder view.
[^77]: https://github.com/jx-admin/Code2/blob/master/androidCode.url — Committed .url specimen: Prop3=19,11, IDList= empty, http favicon IconFile, IconIndex=1.
[^78]: https://stackoverflow.com/questions/62490091 — Stack Overflow specimen: real Explorer-written file with IconFile=https://cdn.sstatic.net/… favicon and non-fixed section order.
[^79]: https://github.com/MakiseKurisu/Win86emu/blob/master/yact/_ReactOS_Dlls/x86node.reg — Registry dump: HKCR\InternetShortcut DefaultIcon = %SystemRoot%\system32\url.dll,0; shell\open\command = rundll32 shdocvw.dll,OpenURL.
[^80]: http://www.metabpa.org/projects/psbrowserbookmarks/about_browserbookmarks — metabpa browser-bookmarks project: reference .url with Roamed=-1; GUID property sections for Description/Notes/Rating; .website vs .url.
[^81]: https://learn.microsoft.com/zh-cn/previous-versions/troubleshoot/browsers/core-features/apply-property-error — Microsoft KB (archived): Description/Notes/Rating serialized as Prop21/Prop5/Prop9 in GUID sections.
[^82]: https://community.sap.com/t5/application-development-discussions/sending-url-as-attachment/m-p/9339383 — SAP community: minimal Web Slice specimen `[MonitoredItem] FeedUrl=http://scn.sap.com/ IsLivePreview=true`.
[^83]: https://github.com/TheTeamAlexa/IDM-Crack-Internet-Download-Manager-6.40/blob/main/Subscribe%20On%20Youtube.website — Committed 2022 .website specimen: `IconFile=https://www.youtube.com/s/desktop/4468d336/img/favicon_32x32.png` (PNG favicon).

<!-- chapter-nav -->

---

← [2. Sections](02-sections.md) · [Chapters](README.md) · [4. Property bags →](04-property-bags.md)
