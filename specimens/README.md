# Specimens

Verbatim `.url` and `.website` files cited throughout the catalog, kept so the byte-level claims in [the chapters](../docs/README.md) can be checked against primary evidence.

The name before `.txt` is the original filename (`androidCode.url.txt` was `androidCode.url`). Line endings here are LF. Real Windows-written files use CRLF, and legacy favorites may be ANSI, modified UTF-7, or UTF-16. See [Chapter 2](../docs/02-sections.md).

> **Safety**
>
> Every specimen in this directory is plain text with a `.txt` suffix, including the benign files. Windows parses a real `.url` or `.website` as soon as Explorer displays the folder, and several benign specimens set `IconFile` to a live favicon URL. Leave the `.txt` suffix in place and open the files in an editor.
>
> Everything under [`poc/`](poc/) is a real in-the-wild malicious specimen. The copies are inert, and they still name real attack infrastructure: attacker SMB and WebDAV servers, staging paths, and LOLBin chains. Leave those hosts alone. Hosts and IPs are defanged with `[.]`, either in the source report or by this repo (noted per file).

## Benign specimens

| File | What it shows | Source |
|---|---|---|
| [androidCode.url.txt](androidCode.url.txt) | Real Explorer-written .url committed to GitHub (jx-admin/Code2). FMTID_Intshcut section with the ubiquitous undocumented Prop3=19,11. | https://github.com/jx-admin/Code2/blob/master/androidCode.url |
| [ie9-pinned-site_accad.website.txt](ie9-pinned-site_accad.website.txt) | Full IE9-era pinned-site .website committed to GitHub: Prop3 + Prop4 (PID_IS_NAME title) in FMTID_Intshcut, plus the A7AF692E pinned-site section (with Prop2 VT_BLOB) and the 9F4C2855 AppUserModel section. | https://github.com/JONGGON/DeepHumanPrediction/blob/master/DeepHumanPrediction/Code/DeepHumanPrediction/Motion_Prediction_Simple/Data/ACCAD/ACCAD%20-%20Motion%20Capture%20Lab%20-%20data%20and%20downloads.website |
| [google-com.website.txt](google-com.website.txt) | Canonical google.com .website (forum-documented): Prop3=19,2 variant, Prop4 title, A7AF692E section with Prop2 blob and Prop6=3,1. | https://community.spiceworks.com/t/microsoft-edge-desktop-shortcut/790553 |
| [bookmarklet_extendedurl.url.txt](bookmarklet_extendedurl.url.txt) | IE bookmarklet favorite (GitHub gist, verbatim): [Bookmarklet] ExtendedURL holds the full javascript: payload (<=5119 bytes) while URL= carries the <=2083-byte truncation. Note the rare Prop3=19,15. | https://gist.github.com/dungsaga/45788260c81832de54ab5e237b523d22 |
| [web-slice_monitoreditem.url.txt](web-slice_monitoreditem.url.txt) | IE8 Web Slice / feed-monitor favorite fragment (SAP community, verbatim): [MonitoredItem] with FeedUrl (lowercase 'rl') and IsLivePreview. | https://community.sap.com/t5/application-development-discussions/sending-url-as-attachment/m-p/9339383 |
| [epic-launcher_fortnite.url.txt](epic-launcher_fortnite.url.txt) | Epic Games Launcher desktop shortcut (Kodi forum, verbatim user paste): custom com.epicgames.launcher:// scheme, WorkingDirectory=, exe as IconFile, Prop3=19,0. | https://forum.kodi.tv/showthread.php?tid=287826&page=120 |
| [steam_rungameid-730.url.txt](steam_rungameid-730.url.txt) | Steam desktop shortcut: steam://rungameid/<appid> scheme with per-game icon hash .ico in the Steam games folder. | https://github.com/Ignefolio/Steam-Shortcut-Icon-Fixer |

## Malicious specimens ([poc/](poc/))

**Inert text copies of weaponized shortcuts — see the safety warning above.**

| File | Campaign / CVE | What it shows | Source |
|---|---|---|---|
| [cve-2024-43451_xd.url.txt](poc/cve-2024-43451_xd.url.txt) | xd.url (SHA1 76e93c97ffdb5adb509c966bca22e12c4508dcaa) | Check Point Research, 'NTLM Exploits Bomb' campaign. CVE-2024-43451-style NTLMv2 hash leak: IconFile on a UNC share forces SMB authentication on right-click/delete/drag. IP already defanged in the source report. | https://research.checkpoint.com/2025/cve-2025-24054-ntlm-exploit-in-the-wild/ |
| [cve-2025-24054_xd.website.txt](poc/cve-2025-24054_xd.website.txt) | xd.website (SHA1 84132ae00239e15b50c1a20126000eed29388100) | same Check Point campaign; a weaponized pinned-site .website with the full property-store sections, pointing at the same SMB server. IP already defanged in the source report. | https://research.checkpoint.com/2025/cve-2025-24054-ntlm-exploit-in-the-wild/ |
| [cve-2025-33053_stealth-falcon_tlm005.pdf.url.txt](poc/cve-2025-33053_stealth-falcon_tlm005.pdf.url.txt) | TLM.005_TELESKOPIK_MAST_HASAR_BILDIRIM_RAPORU.pdf.url | Stealth Falcon / CVE-2025-33053 (Check Point). WorkingDirectory= hijack to an attacker WebDAV share so the LOLBin iediagcmd.exe runs the attacker's route.exe. Domain already defanged in the source report. | https://research.checkpoint.com/2025/stealth-falcon-zero-day/ |
| [cve-2023-36025_docusign3.url.txt](poc/cve-2023-36025_docusign3.url.txt) | DocuSign3.url | Trend Micro, Phemedrone Stealer campaign abusing CVE-2023-36025 (SmartScreen bypass): URL= reaches a .cpl inside a ZIP on a raw-IPv4 WebDAV share via zip-in-path syntax. Note Prop3=19,9. IP defanged by this repo ([.]). | https://www.trendmicro.com/en_us/research/23/k/cve-2023-36025-exploited-for-defense-evasion-in-phemedrone-stealer-campaign.html |
| [cve-2024-21412_water-hydra_photo_2023-12-29.jpg.url.txt](poc/cve-2024-21412_water-hydra_photo_2023-12-29.jpg.url.txt) | photo_2023-12-29.jpg.url | Water Hydra / CVE-2024-21412 first stage (Trend Micro): disguised as a JPEG (imageres.dll icon 126), points at a second .url on a WebDAV share. IP defanged by this repo ([.]). | https://www.trendmicro.com/en_us/research/24/b/cve-2024-21412--internet-shortcut-files-targeted-attack.html |
| [cve-2023-36025_water-hydra_2.url.txt](poc/cve-2023-36025_water-hydra_2.url.txt) | 2.url | Water Hydra second stage (Trend Micro): CVE-2023-36025 zip-in-path logic, URL= points at a2.cmd inside a2.zip on the WebDAV share; Prop3=19,9. IP defanged by this repo ([.]). | https://www.trendmicro.com/en_us/research/24/b/cve-2024-21412--internet-shortcut-files-targeted-attack.html |

## Notes

- Original filenames and IOCs for the malicious copies are in the table above and in [Chapter 5](../docs/05-security-behaviors.md).
- `web-slice_monitoreditem.url.txt` is a verbatim fragment of a larger favorite, as published by the source.
