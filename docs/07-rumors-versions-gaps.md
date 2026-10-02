# 6. Unverified Keys, Version Differences, and Open Gaps

> Part of the [catalog](../README.md). [Chapters](README.md) · [Bibliography](../references.md) · [Specimens](../specimens/README.md)
>
> ← [Abuse cases](06-abuse-cases.md) · **6. Rumors, versions, and gaps**

## Contents

- [6.1 Unverified and rumored keys](#61-unverified-and-rumored-keys)
- [6.2 Windows-version behavior differences](#62-windows-version-behavior-differences)
- [6.3 Open gaps](#63-open-gaps)


This closing chapter quarantines the claims that survived none of the evidence checks ([§6.1](#61-unverified-and-rumored-keys)), records how .url behavior shifts across Windows generations ([§6.2](#62-windows-version-behavior-differences)), and lists the format's documented-nowhere semantics ([§6.3](#63-open-gaps)). Entries here are deliberately *not* merged into chapters 2–4: a key that appears only in a rumor table should never be mistaken for catalog data.

## 6.1 Unverified and rumored keys

Every row below was claimed somewhere as a .url key or behavior and was then hunted across real specimens, Microsoft documentation, SDK headers, and the Wine/ReactOS parser trees. **MotW** (Mark-of-the-Web) and other terms are defined in [chapter 5](05-security-behaviors.md); PROPID/VT tags in [chapter 4](04-property-bags.md).

| Claimed key/section | Claimed meaning | Source of claim | Evidence search result | Verdict |
|---|---|---|---|---|
| `SiteURL=` | Site address key | Search hits only | All hits are PowerShell `$SiteURL` *variables* used to build a normal `URL=` line; zero .url-content hits[^142] | **[Rumor]** — no evidence |
| `ScriptUrl=` / `ScriptURL=` | Script target | — | Zero hits as a .url key; only unrelated JavaScript runtime data (`scriptUrl = script && script.url`) in sandbox memory scans[^143] | **[Rumor]** — no evidence |
| `Referer=` / `Referrer=` | HTTP Referer header to send | filetypedb.com only | `filetype:url "Referer="` returns 0 results; no Referer PROPID exists in the documented FMTID_Intshcut/FMTID_InternetSite sets[^144][^145] | **[Rumor]** — single uncorroborated source |
| `BrowserFlags=` | Browser behavior flags | filetypedb.com | Real, but as a **registry** value under `HKCR\<ProgID>` (e.g. `"BrowserFlags"=dword:00000008` under `HKCR\https`) controlling in-place browsing — registry/file-format cross-wiring[^146][^144] | **[Rumor]** as a .url key; real registry value |
| `BaseURL=` inside `[InternetShortcut]` | Base for relative resolution | filetypedb.com | Every real specimen places `BASEURL=` in `[DEFAULT]` or `[DOC#n…]` (with `ORIGURL=`); the only "[InternetShortcut] + BaseURL" hit is daedalOS, a web-based desktop *emulator*, not Windows[^147][^148] | **[Rumor]** for that placement |
| Negative `IconIndex=` | Resource ID instead of ordinal | cnblogs blog examples (`IconIndex=-48`) | No captured specimen; ss64 states the negative=resource-ID rule explicitly for **Desktop.ini**, not .url; plausible via `ExtractIcon` resource-ID semantics, and malware specimens show only large *positive* indices[^149][^150] | **[Rumor]**-leaning — blog claims only |
| Pinned/taskbar keys in .url | Pin state inside the file | — | No `Pinned=`/`Taskbar=` key in any specimen; pinning is external state (`%AppData%\Microsoft\Internet Explorer\Quick Launch\User Pinned\TaskBar`; IE9 pinned sites use the separate `.website` extension + `iexplore.exe -w "%l" %*`)[^151][^152] | **[Rumor]** — pinning is external state |
| Non-`Prop*` keys in `.website` | Extra named keys | — | All specimens (benign and the malicious `xd.website`) contain only standard `[InternetShortcut]` keys plus `Prop<N>=` lines under GUID sections[^153][^154] | **Busted** — no named keys beyond `Prop*` found |

The pattern is that three of the weakest rows — `Referer=`, `BrowserFlags=`, and `BaseURL=`-in-`[InternetShortcut]` — trace to a single page, filetypedb.com, a format-aggregator that appears partly AI-generated (its reference list includes fabricated-looking citations) and whose claims are contradicted by every specimen set.[^144] It is flagged here as a weak source: nothing it alone asserts appears anywhere in chapters 2–5. The negative-`IconIndex` row is the only rumor with a plausible mechanism — the Win32 `ExtractIcon` API does interpret negative indices as resource IDs, and the rule is confirmed for `Desktop.ini` — but plausibility is not a specimen, so it stays quarantined.[^149][^150] Conversely, two claims investigated in the same sweep were *promoted out* of the rumor table: `IconFile=` with an http(s) URL (confirmed by many real IE favorites) and `[MonitoredItem]` `IsLivePreview`/`PreviewSize` (confirmed by real "Web Slice Gallery.url"/"Suggested Sites.url" specimens); both are cataloged in chapters 2–3.[^155]

## 6.2 Windows-version behavior differences

Because Microsoft has repeatedly *depowered individual fields* rather than documenting the format ([chapter 5, §5.7](05-security-behaviors.md#57-patch-archaeology-and-windows-version-notes)), semantics are version-dependent. **WebDAV** below means Web Distributed Authoring and Versioning, the HTTP-based remote-filesystem protocol Explorer mounts transparently.

| Era | Open command (`HKCR\InternetShortcut\shell\Open\Command`) | `.website` / pinned-site support | `[MonitoredItem]` / Web Slices | `WorkingDirectory=` honored for exe launches | MotW / SmartScreen gating | IE component availability |
|---|---|---|---|---|---|---|
| XP / 2003 (shdocvw era) | `rundll32.exe shdocvw.dll,OpenURL %l`[^156] | No (pre-IE9) | No (pre-IE8) | Yes (ungated) | No MotW dialog on .url open | Full IE6–8 **[Observed]** |
| Vista–8.1 (ieframe era) | `rundll32.exe ieframe.dll,OpenURL %l`; shdocvw's OpenURL forwards to ieframe[^157] | IE9–11: `.website` pinning via `iexplore.exe -w`[^151] | IE8-era feature; real Web Slice specimens[^155] | Yes | **MS16-104 (Sep 2016)** adds a MotW security-dialog gate (`CDownloadUtilities::OpenSafeOpenDialog`) on .url open[^157] | Full IE **[RE'd]** |
| Win10 (IE11 + Edge era) | Same ieframe command[^158] | `.website` still parsed; extended favorite properties (rating/notes) no longer editable via UI as of IE10/Win10, though Explorer still displays them if set in the file[^159] | Feature dead with IE8-era feeds; files still parse | Yes | MotW gate active; CVE-2023-36025 adds a SmartScreen check in `windows.storage.dll`; CVE-2024-21412 gates .url→.url chaining (exact mechanics unpublished; [chapter 5](05-security-behaviors.md)) | IE11 present but secondary to Edge **[Observed]** |
| Win10/11 (Edge-only era) | Same ieframe command[^158] | `.website` pinning mechanism disappears with the IE desktop app; files remain parseable as .url | No | Yes, until June 2025 | CVE-2024-43451 patches render-time NTLM leak (Nov 2024) | IE11 desktop app disabled, redirects to Edge; IE binaries retained for IE mode[^160] **[Documented]** |
| Win11 24H2 | Unchanged per available evidence — ieframe/url.dll/shdocvw + OpenURL persist for IE mode (**inference, Med-High**; no authoritative 24H2 export dump)[^160][^161] | No | No | **No** — CVE-2025-33053 patch (June 10, 2025) makes Windows "ignore the WorkingDirectory value when launching executables"[^162] | MotW + SmartScreen stack as above | `iediagcmd.exe` removed with IE, breaking the original Stealth Falcon hijack (Rapid7); attackers generated 59 alternative LOLBin .url variants[^163] **[Observed]** |

Three rows carry caveats worth restating. The XP→Vista switch of the open command from `shdocvw.dll` to `ieframe.dll` is not a removal: shdocvw's `OpenURL` export still exists and forwards, which is why all three DLLs remain LOLBAS proxy-execution primitives on Windows 10/11.[^161] The 24H2 row is the matrix's softest cell: what is *confirmed* is that `iediagcmd.exe` is gone (Rapid7's verbatim test-kit note) and that Microsoft services the IE binaries through at least 2029 for IE mode; the persistence of `ieframe!OpenURL` and the registry association is inferred from that servicing commitment plus LOLBAS's continued "Windows 10, Windows 11" listing, not from a verified 24H2 export listing.[^160][^163] Finally, the WorkingDirectory column shows the sharpest single-field behavior change in the format's history: honored unconditionally for two decades, then neutered OS-wide by one patch — so any statement about `WorkingDirectory=` must be dated.[^162]

## 6.3 Open gaps

The following items are referenced in documentation, headers, or real files, but their semantics are documented nowhere. Each is a concrete target for future reverse engineering.

| Item | Where referenced | What is unknown | Confidence |
|---|---|---|---|
| `Prop3` semantics (FMTID_Intshcut, PID 3) | Real files worldwide; values observed 0, 2, 9, 11, 15 (VT_UI4)[^164][^165] | Meaning of the value; why exploit specimens converge on `19,9` while benign files carry `19,0/2/11`; PID 3 is absent from the public PID_IS_* list (it jumps 2→4)[^145] | [Observed] (existence; semantics unknown) |
| `Prop2` VT_BLOB in `{A7AF692E-…}` (.website) | Check Point `xd.website` + three benign .website specimens[^154][^153] | Full structure. Partial analysis (this catalog): all four specimens are exactly 45 bytes, share the prefix `2C000000 00000000 01000000` + 16 bytes of `FF`, then four varying DWORDs and one varying trailing byte; the leading `0x2C` (44) looks like a length prefix and the trailing byte checksum-like, but no field assignment is established | [Observed] (specimens); structural reading is catalog inference |
| `IDList=` semantics | Present, almost always empty, in IE-written favorites[^166] | What shell items it encodes when populated; the third-party rclip parser notes the encoding would be `WritePrivateProfileStruct` hex plus a checksum byte, but no populated specimen was found[^166] | [Observed] |
| FMTID_InternetSite `{000214A1-…}` on disk | SDK `ShlGuid.h` and the Learn page define it and its PID_INTSITE_* table[^167][^145] | Never observed serialized in any real .url file; Wine/ReactOS reference it only in headers and a CLSID round-trip test, never in a parser[^168] | [Documented] (defined in SDK/Learn; never observed on disk) |
| `PID_INTSITE_RAWURL` | `ShlObj.h` comment: "The raw, un-encoded, unicode url"[^167] | No `#define` exists in the SDK header and it is absent from the Learn page — its PROPID value is unknown | [Documented] (comment only) |
| Blake HotKey table, digit block | "Unofficial Guide" Appendix A (both 2nd and 3rd editions)[^165] | Of the 284 values the guide publishes, the 30 digit-block entries in the C+S, S+A and C+S+A columns are shifted by one for every digit 0–9: the runs 817–826, 1329–1338 and 1841–1850 carry low bytes 0x31–0x3A where the correct digit VKs are 0x30–0x39 (e.g. Ctrl+Shift+0 printed as 817 where the formula gives 816); only the C+A column (1584–1593) is correct. An off-by-one in the source, so the printed digit encodings are unverified against Windows itself — treat the formula, not the table, as authoritative ([§3.4](03-keys.md#34-the-hotkey-encoding)) | [Observed] (anomaly in source) |
| `ShowCommand=1` | vpscoder lists 1/3/7; Blake lists only absent/3/7[^169][^165] | 1 = SW_NORMAL is plausible and treated as default-by-absence by the rclip parser, but no real specimen with `ShowCommand=1` was captured[^166] | [Observed] (no specimen) |
| `.W` UTF-7 encoder/decoder ownership | Specimens prove `[InternetShortcut.W]` is modified UTF-7 (`Proc+AOk-dure` = `Procédure`)[^170] | Which exact routine in ieframe (or elsewhere) performs the encode/decode, and its escaping rules at the byte level — tatsu-syo's RE describes `+`→`+-`, `-`→`-+` escaping but does not name the owning function[^171] | [Observed] (encoding); ownership unknown |
| `Modified=` checksum byte | Blake calls the 9th byte "a checksum and … unimportant"; rclip hands it back raw, unvalidated[^165][^166] | The checksum algorithm exists (the byte varies systematically) but its formula is undocumented | [Observed] |
| `Prop3=19,9` functional role | MS16-104 PoC, CVE-2023-36025 and CVE-2024-21412 specimens ([chapter 5](05-security-behaviors.md)) | Whether the value has *any* functional role in exploit specimens, or is simply an artifact of files authored/saved through a common tool — Quarkslab's binary analysis assigns it no exploit role[^157] | [Observed] (recurrence); role unknown |

Two of these gaps are cheap to close and high-value: the `Prop2` blob, because four independent 45-byte specimens already exist and differ only in a 17-byte tail, making differential analysis feasible; and the `Modified=` checksum, because generating .url files with known timestamps via `IUniformResourceLocator` and correlating the ninth byte would recover the formula in a single experiment. The `Prop3` gap is the most security-relevant: a value that appears in three generations of exploit specimens yet is explained by no source — Microsoft, community, or reverse engineer — remains the format's outstanding mystery, and this catalog deliberately records it as *undocumented PROPID 3* rather than assigning it a meaning.[^164][^157]

---

## Footnotes

[^142]: https://www.sharepointdiary.com/2020/11/how-to-add-a-link-to-sharepoint-online-document-library.html — SharePoint Diary (PowerShell `$SiteURL` variable building a standard `URL=` .url file).
[^143]: https://hybrid-analysis.com/sample/1de9fe7617cf2fb402a926386de7c79e9fc800a0a712a76160e69066f7a54c0f/69015e88ca537e92870dd382 — Hybrid Analysis sandbox memory scan (only `scriptUrl` JavaScript runtime hits; no .url key).
[^144]: https://filetypedb.com/web/url — filetypedb.com, "URL File Format — Internet Shortcut" (sole source for Referer/BrowserFlags/BaseURL-in-[InternetShortcut]; flagged weak/partly AI-generated).
[^145]: https://learn.microsoft.com/en-us/windows/win32/lwef/internet-shortcuts — Microsoft Learn, "Internet Shortcuts" (documented PID_IS_*/PID_INTSITE_* sets; no Referer, BrowserFlags, or PID 3).
[^146]: https://blog.51cto.com/qicaiiwang/432259 — registry dump showing `"BrowserFlags"=dword:00000008` under `HKCR\https` (registry value, not a .url key).
[^147]: https://ubuntugenius.wordpress.com/2009/12/09/how-to-open-url-internet-explorer-shortcuts-in-ubuntu-using-firefox/ — real IE8-generated favorite with `BASEURL=` in `[DEFAULT]`; https://blog.51cto.com/u_12197/11240148 — `[DOC#4#5] BASEURL=/ORIGURL=` frameset persistence specimen.
[^148]: https://github.com/DustinBrett/daedalOS/discussions/230 — daedalOS (web desktop emulator) discussion; only "[InternetShortcut] + BaseURL" hit, not real Windows.
[^149]: https://www.cnblogs.com/suv789/p/18324691 — cnblogs article showing `IconIndex=-48`/`-8` examples ("negative index means resource ID, not ordinal"); blog-level evidence only.
[^150]: https://ss64.com/ps/syntax-shortcut.html — ss64: negative IconIndex = resource ID rule stated for Desktop.ini, not .url; https://www.forcepoint.com/blog/x-labs/remcos-malware-new-face — malware specimens with large positive IconIndex values only.
[^151]: https://www.zdnet.com/article/ie9-power-tips-the-secrets-of-pinned-site-shortcuts/ — ZDNet, "IE9 power tips: the secrets of pinned site shortcuts" (`.website` extension invokes `iexplore.exe -w "%l" %*`).
[^152]: https://winaero.com/how-to-use-ie-pinned-sites-on-taskbar-without-disabling-addons/ — Winaero (taskbar pins stored under `User Pinned\TaskBar`; pinning is external state).
[^153]: https://www.autoitscript.com/forum/topic/150926-internet-shortcut-sanitizer-url-and-website-files/ — AutoIt forum, full verbatim .website specimen (only standard keys + Prop lines across three GUID sections).
[^154]: https://research.checkpoint.com/2025/cve-2025-24054-ntlm-exploit-in-the-wild/ — Check Point Research, "CVE-2025-24054 NTLM Exploit in the Wild" (verbatim malicious `xd.website` with the 45-byte Prop2 VT_BLOB).
[^155]: https://hybrid-analysis.com/sample/db1696106bb100a1fc10fadc9b93e17f80055604fa53fe3785b64efdabe0a254/56af7da60e316d585ed41a73 — real dropped "Web Slice Gallery.url"/"Suggested Sites.url" with `[MonitoredItem] IsLivePreview/PreviewSize`; http://forum.hotfix.pl/problemy/problem-ze-skrotem-uslugi-sugerowane-witryny-w-ie-t21332.html — full "Suggested Sites.url" specimen.
[^156]: https://www.schalley.eu/2009/10/05/restore-windows-file-associations/ — XP-era file-association restore .reg (`shdocvw.dll,OpenURL %l`); corroborated by malware registry analyses.
[^157]: https://blog.quarkslab.com/analysis-of-ms16-104-url-files-security-feature-bypass-cve-2016-3353.html — Quarkslab, "Analysis of MS16-104: .URL files Security Feature Bypass (CVE-2016-3353)" (ieframe OpenURL command; MS16-104 MotW dialog gate; Prop3=19,9 assigned no exploit role).
[^158]: https://www.tenforums.com/general-support/193229-windows-file-explorer-wont-execute-url-shortcuts-3.html — Windows 11 registry dump (`ieframe.dll,OpenURL %l`); https://github.com/rcmaehl/MSEdgeRedirect/issues/171 — Windows 11 21H2 confirmation.
[^159]: http://www.metabpa.org/projects/psbrowserbookmarks/about_browserbookmarks — metabpa browser-bookmarks project ("As of IE10 and Windows 10, editing a Favorite's extended properties … doesn't seem to be possible. Explorer will display those parameters correctly, however, if they have been set in the file.").
[^160]: https://techcommunity.microsoft.com/blog/windows-itpro-blog/internet-explorer-11-desktop-app-retirement-faq/2366549 — Microsoft, "Internet Explorer 11 desktop app retirement FAQ" (IE binaries retained and serviced for IE mode through at least 2029).
[^161]: https://lolbas-project.github.io/lolbas/Libraries/Ieframe/ — LOLBAS Ieframe.dll (OpenURL valid on "Windows 10, Windows 11"); companion entries: /lolbas/Libraries/Url/ and /lolbas/Libraries/Shdocvw/.
[^162]: https://blog.0patch.com/2025/06/micropatches-released-for-webdav-remote.html — 0patch, "Micropatches Released for WebDAV Remote Code Execution (CVE-2025-33053)" (verbatim: Microsoft "chang[es] the behavior of URL files such as to ignore the WorkingDirectory value when launching executables").
[^163]: https://www.rapid7.com/blog/post/tr-exposed-webdav-malware-delivery-lab-analysis/ — Rapid7, analysis of exposed attacker WebDAV test server (Win11 24H2 IE removal breaks iediagcmd hijack; 59 alternative .url LOLBin variants).
[^164]: http://www.metabpa.org/projects/psbrowserbookmarks/about_browserbookmarks — metabpa (`Prop3=19,11` shown without gloss); https://arstechnica.com/civis/threads/whats-the-trick-for-creating-large-icons-for-xp.185275/ — Ars Technica thread ("I also don't know what the Prop3= key is for"); https://gist.github.com/596dd5645906cef4716af4abf58df661 — Steam shortcut gist (`Prop3=19,0`).
[^165]: https://www.cyanwerks.com/formats/file-format-url.html — Edward L. Blake, "An Unofficial Guide to the URL File Format" (HotKey Appendix A with the digit-block off-by-one anomaly in both editions; Modified checksum byte "unimportant"; ShowCommand absent/3/7 only).
[^166]: https://docs.rs/rclip-url-file/latest/rclip_url_file/ — rclip-url-file crate documentation (IDList "almost always empty … WritePrivateProfileStruct hex plus a checksum byte"; Modified checksum "handed back raw rather than validated"; ShowCommand raw-number handling).
[^167]: https://github.com/tpn/winsdk-10/blob/master/Include/10.0.10240.0/um/ShlObj.h — Windows SDK 10.0.10240.0 ShlObj.h (PID_INTSITE_RAWURL named in a comment with no #define); ShlGuid.h in the same tree defines FMTID_InternetSite.
[^168]: https://github.com/wine-mirror/wine — Wine source tree (tree-wide grep: FMTID_InternetSite only in include/shlguid.h and a shlwapi CLSID test; no parser reference; same result in ReactOS).
[^169]: https://fileformat.vpscoder.com/task-753 — fileformat.vpscoder.com, "Task 753: .URL File Format" (ShowCommand "1 for normal, 3 for maximized, 7 for minimized").
[^170]: https://learn.microsoft.com/en-us/answers/questions/2150484/extremely-slow-open-file-dialog-from-all-applicati — Microsoft Q&A specimen proving `.A` = ANSI / `.W` = UTF-7 (`Procédure` / `Proc+AOk-dure`).
[^171]: https://www.tatsu-syo.info/Devroom/IEfavorites.html — tatsu-syo.info reverse engineering of IE Favorites (modified-UTF-7 `.W` encoding with `+`→`+-`, `-`→`-+` escaping; owning function not identified).

<!-- chapter-nav -->

---

← [Abuse cases](06-abuse-cases.md) · [Chapters](README.md)
