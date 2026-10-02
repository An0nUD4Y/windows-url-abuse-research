# Bibliography

Each work is listed once. Chapter footnotes keep the original numbers (`[^52]` and the rest), and one work often has several numbers because the chapters were split from a single draft. Look a number up in the [citation index](#citation-number-index).

Chapters 1–6 share this numbering. [Abuse cases](docs/06-abuse-cases.md) numbers its footnotes from 1 again, inside that file only. Sources that appear only there are [listed at the end](#sources-used-only-in-abuse-cases).

Compiled 2026-09-13. Confidence labels are defined in [Chapter 1](docs/01-introduction.md#13-evidence-sources-and-confidence-model).

## Microsoft documentation

- <a id="microsoft-learn-internet-shortcuts"></a>**Microsoft Learn, "Internet Shortcuts".** [https://learn.microsoft.com/en-us/windows/win32/lwef/internet-shortcuts](https://learn.microsoft.com/en-us/windows/win32/lwef/internet-shortcuts). (CLSID_InternetShortcut, IUniformResourceLocator, IPropertySetStorage usage). Cited as 4, 19, 52, 85, 114, 145. Used in Ch. 1, Ch. 2, Ch. 3, Ch. 4, Ch. 5, Ch. 6.

- <a id="microsoft-learn-sample-url-launcher-shortcut"></a>**Microsoft Learn, sample .URL launcher shortcut.** [https://learn.microsoft.com/en-us/windows/mixed-reality/distribute/implementing-3d-app-launchers-win32](https://learn.microsoft.com/en-us/windows/mixed-reality/distribute/implementing-3d-app-launchers-win32). (only MS-published INI serialization example). Cited as 8, 22, 88. Used in Ch. 1, Ch. 2, Ch. 4.

- <a id="microsoft-q-a-2150484"></a>**Microsoft Q&A 2150484.** [https://learn.microsoft.com/en-us/answers/questions/2150484/extremely-slow-open-file-dialog-from-all-applicati](https://learn.microsoft.com/en-us/answers/questions/2150484/extremely-slow-open-file-dialog-from-all-applicati). (Procédure / Proc+AOk-dure .A/.W specimen). Cited as 13, 23, 109, 170. Used in Ch. 1, Ch. 2, Ch. 5, Ch. 6.

- <a id="microsoft-learn-system-appusermodel-id"></a>**Microsoft Learn, System.AppUserModel.ID.** [https://learn.microsoft.com/en-us/windows/win32/properties/props-system-appusermodel-id](https://learn.microsoft.com/en-us/windows/win32/properties/props-system-appusermodel-id). (formatID 9F4C2855-…, propID 5). Cited as 32, 87. Used in Ch. 2, Ch. 4.

- <a id="microsoft-learn-ie11-deployment-guide"></a>**Microsoft Learn, IE11 deployment guide.** [https://learn.microsoft.com/en-us/previous-versions/windows/internet-explorer/ie-it-pro/internet-explorer-11/ie11-deploy-guide/deploy-pinned-sites-using-mdt-2013](https://learn.microsoft.com/en-us/previous-versions/windows/internet-explorer/ie-it-pro/internet-explorer-11/ie11-deploy-guide/deploy-pinned-sites-using-mdt-2013). ("a .website file is like a shortcut…"). Cited as 33. Used in Ch. 2.

- <a id="microsoft-kb"></a>**Microsoft KB.** [https://learn.microsoft.com/zh-cn/previous-versions/troubleshoot/browsers/core-features/apply-property-error](https://learn.microsoft.com/zh-cn/previous-versions/troubleshoot/browsers/core-features/apply-property-error). (archived): Description/Notes/Rating serialized as Prop21/Prop5/Prop9 in GUID sections. Cited as 38, 81, 93. Used in Ch. 2, Ch. 3, Ch. 4.

- <a id="microsoft-q-a"></a>**Microsoft Q&A.** [https://learn.microsoft.com/ja-jp/answers/questions/4115480/question-4115480](https://learn.microsoft.com/ja-jp/answers/questions/4115480/question-4115480). (ja): full property-set dump incl. {F29F85E0} SummaryInformation and .A/.W GUID-section shadows. Cited as 40, 94. Used in Ch. 2, Ch. 4.

- <a id="microsoft-security-update-guide-cve-2024-21412"></a>**Microsoft Security Update Guide, CVE-2024-21412.** [https://msrc.microsoft.com/update-guide/vulnerability/CVE-2024-21412](https://msrc.microsoft.com/update-guide/vulnerability/CVE-2024-21412). (CVSS 8.1, CWE-693, Feb 13 2024); CVE-2024-29988 follow-on per Trend Micro "Facts and Fixes". Cited as 134. Used in Ch. 5.

- <a id="microsoft-internet-explorer-11-desktop-app-retirement-faq"></a>**Microsoft, "Internet Explorer 11 desktop app retirement FAQ".** [https://techcommunity.microsoft.com/blog/windows-itpro-blog/internet-explorer-11-desktop-app-retirement-faq/2366549](https://techcommunity.microsoft.com/blog/windows-itpro-blog/internet-explorer-11-desktop-app-retirement-faq/2366549). (IE binaries retained and serviced for IE mode). Cited as 141, 160. Used in Ch. 5, Ch. 6.

## Windows SDK headers

- <a id="windows-10-sdk-shlguid-h"></a>**Windows 10 SDK ShlGuid.h.** [https://github.com/tpn/winsdk-10/blob/master/Include/10.0.10240.0/um/ShlGuid.h](https://github.com/tpn/winsdk-10/blob/master/Include/10.0.10240.0/um/ShlGuid.h). (FMTID_Intshcut / FMTID_InternetSite GUID definitions). Cited as 5, 21, 96. Used in Ch. 1, Ch. 2, Ch. 4.

- <a id="windows-10-sdk-shlobj-h"></a>**Windows 10 SDK ShlObj.h.** [https://github.com/tpn/winsdk-10/blob/master/Include/10.0.10240.0/um/ShlObj.h](https://github.com/tpn/winsdk-10/blob/master/Include/10.0.10240.0/um/ShlObj.h). (PID_IS_* numeric PROPIDs and variant types). Cited as 6, 20, 53, 86, 167. Used in Ch. 1, Ch. 2, Ch. 3, Ch. 4, Ch. 6.

## Format guides and parsers

- <a id="edward-l-blake-an-unofficial-guide-to-the-url-file-format"></a>**Edward L. Blake, "An Unofficial Guide to the URL File Format".** [https://www.cyanwerks.com/formats/file-format-url.html](https://www.cyanwerks.com/formats/file-format-url.html). (CRLF + ANSI layout). Cited as 1, 18, 56, 115, 165. Used in Ch. 1, Ch. 2, Ch. 3, Ch. 5, Ch. 6.

- <a id="rclip-url-file-crate-docs"></a>**rclip-url-file crate docs.** [https://docs.rs/rclip-url-file/latest/rclip_url_file/](https://docs.rs/rclip-url-file/latest/rclip_url_file/). (no-specification statement; UTF-8/BOM handling; Wine basis). Cited as 2, 46, 47, 59, 60, 166. Used in Ch. 1, Ch. 2, Ch. 3, Ch. 6.

- <a id="nsis-wiki-creating-internet-shortcuts"></a>**NSIS wiki, "Creating internet shortcuts".** [https://nsis.sourceforge.io/Creating_internet_shortcuts](https://nsis.sourceforge.io/Creating_internet_shortcuts). (unofficial key/section list). Cited as 3, 17, 54, 84. Used in Ch. 1, Ch. 2, Ch. 3, Ch. 4.

- <a id="wine-ieframe-intshcut-c"></a>**Wine ieframe intshcut.c.** [https://github.com/wine-mirror/wine/blob/master/dlls/ieframe/intshcut.c](https://github.com/wine-mirror/wine/blob/master/dlls/ieframe/intshcut.c). (reads only URL/iconfile/iconindex; GetPrivateProfileStringW semantics). Cited as 15, 55. Used in Ch. 1, Ch. 3.

- <a id="wine-ieframe-test-fixture"></a>**Wine ieframe test fixture.** [https://github.com/wine-mirror/wine/blob/master/dlls/ieframe/tests/intshcut.c](https://github.com/wine-mirror/wine/blob/master/dlls/ieframe/tests/intshcut.c). (Prop0=1,2 loaded then asserted ignored). ReactOS equivalent: https://github.com/reactos/reactos/blob/master/dll/win32/ieframe/intshcut.c. Cited as 16, 67, 89. Used in Ch. 1, Ch. 3, Ch. 4.

- <a id="tatsu-syo-info"></a>**tatsu-syo.info.** [https://www.tatsu-syo.info/Devroom/IEfavorites.html](https://www.tatsu-syo.info/Devroom/IEfavorites.html). (Japanese RE page: .W = modified UTF-7, +→+- / -→-+ escaping). Cited as 25, 105, 171. Used in Ch. 2, Ch. 4, Ch. 6.

- <a id="metabpa-browser-bookmarks-project"></a>**metabpa browser-bookmarks project.** [https://www.metabpa.org/projects/psbrowserbookmarks/about_browserbookmarks](https://www.metabpa.org/projects/psbrowserbookmarks/about_browserbookmarks). ("As of IE10 and Windows 10, editing a Favorite's extended properties … doesn't seem to be possible. Explorer will display those parameters correctly, however, if they have been set in the file."). Cited as 39, 80, 92, 159, 164. Used in Ch. 2, Ch. 3, Ch. 4, Ch. 6.

- <a id="voidtools-everything-docs"></a>**voidtools Everything docs.** [https://www.voidtools.com/support/everything/properties/](https://www.voidtools.com/support/everything/properties/). (property-set serialization syntax, {F29F85E0} and {64440492} examples). Cited as 41, 95. Used in Ch. 2, Ch. 4.

- <a id="the-broken-event-blog"></a>**The Broken Event Blog.** [https://brokenevent.com/blog/2018-08-26](https://brokenevent.com/blog/2018-08-26). HotKey as bitwise OR of System.Windows.Forms.Keys with Shift=1<<8, Ctrl=1<<9, Alt=1<<10 IconIndex "meaningless without the IconFile.". Cited as 57. Used in Ch. 3.

- <a id="ctrl-blog-what-is-the-best-file-format-for-web-shortcuts"></a>**ctrl.blog, "What is the best file format for web shortcuts".** [https://ctrl.blog/entry/internet-shortcut-files.html](https://ctrl.blog/entry/internet-shortcut-files.html). Title=/Desc= "aren't displayed or used for anything in any modern operating system.". Cited as 68. Used in Ch. 3.

- <a id="blake-2nd-ed"></a>**Blake, 2nd ed.** [https://web.archive.org/web/20151224053029/http://www.lyberty.com/encyc/articles/tech/dot_url_format_-_an_unofficial_guide.html](https://web.archive.org/web/20151224053029/http://www.lyberty.com/encyc/articles/tech/dot_url_format_-_an_unofficial_guide.html). (lyberty mirror via Wayback): pre-FILETIME Modified analysis, Mat Kramer email (29 Aug 2000), same digit-block HotKey anomaly. Cited as 74. Used in Ch. 3.

- <a id="ss64-negative-iconindex-resource-id-rule-stated-for-desktop-ini-"></a>**ss64: negative IconIndex = resource ID rule stated for Desktop.ini, not .url.** [https://ss64.com/ps/syntax-shortcut.html](https://ss64.com/ps/syntax-shortcut.html). Cited as 150. Used in Ch. 6.

- <a id="wine-source-tree"></a>**Wine source tree.** [https://github.com/wine-mirror/wine](https://github.com/wine-mirror/wine). (tree-wide grep: FMTID_InternetSite only in include/shlguid.h and a shlwapi CLSID test; no parser reference; same result in ReactOS). Cited as 168. Used in Ch. 6.

## Vulnerability research

- <a id="ncc-group-the-case-of-missing-file-extensions"></a>**NCC Group, "The Case of Missing File Extensions".** [https://www.nccgroup.com/research/the-case-of-missing-file-extensions/](https://www.nccgroup.com/research/the-case-of-missing-file-extensions/). (NeverShowExt list includes .URL and .website). Cited as 9. Used in Ch. 1.

- <a id="quarkslab-cve-2016-3353-analysis"></a>**Quarkslab, CVE-2016-3353 analysis.** [https://blog.quarkslab.com/analysis-of-ms16-104-url-files-security-feature-bypass-cve-2016-3353.html](https://blog.quarkslab.com/analysis-of-ms16-104-url-files-security-feature-bypass-cve-2016-3353.html). (ieframe OpenURL; no exploit role for Prop3). Cited as 11, 31, 99, 110, 157. Used in Ch. 1, Ch. 2, Ch. 4, Ch. 5, Ch. 6.

- <a id="check-point-research-blind-eagle"></a>**Check Point Research, "Blind Eagle".** [https://research.checkpoint.com/2025/blind-eagle-and-justice-for-all/](https://research.checkpoint.com/2025/blind-eagle-and-justice-for-all/). (in-the-wild .A/.W parser-differential specimen, +AF8- UTF-7). Cited as 14, 24, 100, 116. Used in Ch. 1, Ch. 2, Ch. 4, Ch. 5.

- <a id="check-point-research-cve-2025-24054"></a>**Check Point Research, CVE-2025-24054.** [https://research.checkpoint.com/2025/cve-2025-24054-ntlm-exploit-in-the-wild/](https://research.checkpoint.com/2025/cve-2025-24054-ntlm-exploit-in-the-wild/). (malicious .website with all three GUID sections). Cited as 35, 70, 90, 112, 154. Used in Ch. 2, Ch. 3, Ch. 4, Ch. 5, Ch. 6.

- <a id="check-point-research-stealth-falcon-zero-day"></a>**Check Point Research, "Stealth Falcon Zero-Day".** [https://research.checkpoint.com/2025/stealth-falcon-zero-day/](https://research.checkpoint.com/2025/stealth-falcon-zero-day/). (CVE-2025-33053; verbatim .url; iediagcmd bare-name search-order hijack; Horus chain; CustomShellHost.exe). Cited as 111. Used in Ch. 5.

- <a id="kaspersky-securelist-ntlm-abuse-in-2025"></a>**Kaspersky Securelist, "NTLM abuse in 2025".** [https://securelist.com/ntlm-abuse-in-2025/118132/](https://securelist.com/ntlm-abuse-in-2025/118132/). (Blind Eagle port-80 WebDAV fallback leaking NTLM hashes). Cited as 122. Used in Ch. 5.

- <a id="0patch-micropatches-released-for-webdav-remote-code-execution-cv"></a>**0patch, "Micropatches Released for WebDAV Remote Code Execution (CVE-2025-33053)".** [https://blog.0patch.com/2025/06/micropatches-released-for-webdav-remote.html](https://blog.0patch.com/2025/06/micropatches-released-for-webdav-remote.html). (verbatim patch-mechanism quote). Cited as 123, 162. Used in Ch. 5, Ch. 6.

- <a id="rapid7-analysis-of-exposed-attacker-webdav-test-server"></a>**Rapid7, analysis of exposed attacker WebDAV test server.** [https://www.rapid7.com/blog/post/tr-exposed-webdav-malware-delivery-lab-analysis/](https://www.rapid7.com/blog/post/tr-exposed-webdav-malware-delivery-lab-analysis/). (Win11 24H2 iediagcmd removal; 59 alternative .url LOLBin variants; attacker patch-check notes). Cited as 124, 163. Used in Ch. 5, Ch. 6.

- <a id="google-project-zero-693"></a>**Google Project Zero #693.** [https://bugs.chromium.org/p/project-zero/issues/detail?id=693](https://bugs.chromium.org/p/project-zero/issues/detail?id=693). (Tavis Ormandy; zip-as-folder .hta MotW-evasion trick). Cited as 125. Used in Ch. 5.

- <a id="itresit-labs"></a>**itresit labs.** [https://labs.itresit.es/2026/03/25/when-bills-come-with-surprise-donut-of-python-and-rat/](https://labs.itresit.es/2026/03/25/when-bills-come-with-surprise-donut-of-python-and-rat/). (verbatim DavWWWRoot .wsh .url); Proofpoint Aug-2024 chain summarized at research.kr-labs.com.ua. Cited as 127. Used in Ch. 5.

- <a id="trend-micro-zdi-cve-2024-21412-water-hydra-targets-traders"></a>**Trend Micro ZDI, "CVE-2024-21412: Water Hydra Targets Traders".** [https://www.trendmicro.com/en_us/research/24/b/cve202421412-water-hydra-targets-traders-with-windows-defender-s.html](https://www.trendmicro.com/en_us/research/24/b/cve202421412-water-hydra-targets-traders-with-windows-defender-s.html). (verbatim .url chain; search: AQS lure; SmartScreen/MotW failure). Cited as 128. Used in Ch. 5.

- <a id="zdi-16-506-advisory"></a>**ZDI-16-506 advisory.** [https://www.zerodayinitiative.com/advisories/ZDI-16-506/](https://www.zerodayinitiative.com/advisories/ZDI-16-506/). (credit Eduardo Braun Prado; disclosure timeline). Cited as 130. Used in Ch. 5.

- <a id="mirror-of-m01n-team-analysis-of-cve-2023-36025"></a>**mirror of M01N Team analysis of CVE-2023-36025.** [https://cn-sec.com/archives/2321455.html](https://cn-sec.com/archives/2321455.html). (CInvokeCreateProcessVerb::ProcessCommandTemplate; missing CheckSmartScreenWithAltFile branch). Cited as 131. Used in Ch. 5.

- <a id="trend-micro-cve-2023-36025-exploited-for-defense-evasion-in-phem"></a>**Trend Micro, "CVE-2023-36025 Exploited for Defense Evasion in Phemedrone Stealer Campaign".** [https://www.trendmicro.com/en_us/research/24/a/cve-2023-36025-exploited-for-defense-evasion-in-phemedrone-steal.html](https://www.trendmicro.com/en_us/research/24/a/cve-2023-36025-exploited-for-defense-evasion-in-phemedrone-steal.html). (DocuSign3.url verbatim). Cited as 132. Used in Ch. 5.

- <a id="trend-micro-darkgate-operators-exploit-microsoft-windows-smartsc"></a>**Trend Micro, "DarkGate Operators Exploit Microsoft Windows SmartScreen Bypass".** [https://www.trendmicro.com/en_us/research/24/c/cve-2024-21412--darkgate-operators-exploit-microsoft-windows-sma.html](https://www.trendmicro.com/en_us/research/24/c/cve-2024-21412--darkgate-operators-exploit-microsoft-windows-sma.html). (verbatim DarkGate .url pair). Cited as 133. Used in Ch. 5.

## Specimens and primary files

- <a id="committed-url-specimen-with-000214a0-prop3-19-11"></a>**committed .url specimen with [{000214A0-…}] Prop3=19,11.** [https://github.com/jx-admin/Code2/blob/master/androidCode.url](https://github.com/jx-admin/Code2/blob/master/androidCode.url). Cited as 7, 77. Used in Ch. 1, Ch. 3.

- <a id="registry-dump-of-hkcr-internetshortcut"></a>**registry dump of HKCR\InternetShortcut.** [https://github.com/MakiseKurisu/Win86emu/blob/master/yact/_ReactOS_Dlls/x86node.reg](https://github.com/MakiseKurisu/Win86emu/blob/master/yact/_ReactOS_Dlls/x86node.reg). (NeverShowExt, IsShortcut, shdocvw OpenURL command). Cited as 10, 79. Used in Ch. 1, Ch. 3.

- <a id="ie-bookmarklet-url-gist"></a>**IE bookmarklet .url gist.** [https://gist.github.com/dungsaga/45788260c81832de54ab5e237b523d22](https://gist.github.com/dungsaga/45788260c81832de54ab5e237b523d22). (Prop3=19,15; ExtendedURL). Cited as 27, 64, 102. Used in Ch. 2, Ch. 3, Ch. 4.

- <a id="hybrid-analysis-sandbox"></a>**Hybrid Analysis sandbox.** [https://hybrid-analysis.com/sample/db1696106bb100a1fc10fadc9b93e17f80055604fa53fe3785b64efdabe0a254/56af7da60e316d585ed41a73](https://hybrid-analysis.com/sample/db1696106bb100a1fc10fadc9b93e17f80055604fa53fe3785b64efdabe0a254/56af7da60e316d585ed41a73). Microsoft-shipped Web Slice .url files with [MonitoredItem] FeedUrl=https://ieonline.microsoft.com/#ieslice ieframe.dll template strings FEEDURL/FeedViewer/MonitoredItem. Cited as 29, 65, 155. Used in Ch. 2, Ch. 3, Ch. 6.

- <a id="exiftool-test-suite-url-specimen-with-all-metadata-keys-populate"></a>**ExifTool test-suite .url specimen with all metadata keys populated.** [https://raw.githubusercontent.com/exiftool/exiftool/master/t/images/LNK.url](https://raw.githubusercontent.com/exiftool/exiftool/master/t/images/LNK.url). (Author/WhatsNew/Comment/Desc/Roamed=1/IDList/Modified/ShowCommand=3/HotKey=1582). Cited as 43, 62. Used in Ch. 2, Ch. 3.

- <a id="stack-overflow-specimen"></a>**Stack Overflow specimen.** [https://stackoverflow.com/questions/62490091](https://stackoverflow.com/questions/62490091). (Explorer-written file, Prop4=31,<page title>, section order 000214A0 → A7AF692E → InternetShortcut → 9F4C2855). Cited as 48, 78, 104. Used in Ch. 2, Ch. 3, Ch. 4.

- <a id="cve-2024-43451-poc-url-gist"></a>**CVE-2024-43451 PoC .url gist.** [https://gist.github.com/milo2012/00856e9273ab08829dc715a845abb4ed](https://gist.github.com/milo2012/00856e9273ab08829dc715a845abb4ed). HotKey=0, IconFile=C:\Windows\System32\SHELL32.dll, .A/.W section trick. Cited as 51, 75, 117. Used in Ch. 2, Ch. 3, Ch. 5.

- <a id="committed-2022-website-specimen"></a>**Committed 2022 .website specimen.** [https://github.com/TheTeamAlexa/IDM-Crack-Internet-Download-Manager-6.40/blob/main/Subscribe%20On%20Youtube.website](https://github.com/TheTeamAlexa/IDM-Crack-Internet-Download-Manager-6.40/blob/main/Subscribe%20On%20Youtube.website). IconFile=https://www.youtube.com/s/desktop/4468d336/img/favicon_32x32.png (PNG favicon). Cited as 83. Used in Ch. 3.

- <a id="steam-shortcut-gist"></a>**Steam shortcut gist.** [https://gist.github.com/596dd5645906cef4716af4abf58df661](https://gist.github.com/596dd5645906cef4716af4abf58df661). (Prop3=19,0). Cited as 97. Used in Ch. 4.

- <a id="committed-url-specimen"></a>**committed .url specimen.** [https://github.com/kanalstrahlen/precisION/blob/main/Tutorial2.url](https://github.com/kanalstrahlen/precisION/blob/main/Tutorial2.url). (Prop3=19,11). Cited as 101. Used in Ch. 4.

- <a id="hybrid-analysis-sandbox-memory-scan"></a>**Hybrid Analysis sandbox memory scan.** [https://hybrid-analysis.com/sample/1de9fe7617cf2fb402a926386de7c79e9fc800a0a712a76160e69066f7a54c0f/69015e88ca537e92870dd382](https://hybrid-analysis.com/sample/1de9fe7617cf2fb402a926386de7c79e9fc800a0a712a76160e69066f7a54c0f/69015e88ca537e92870dd382). (only scriptUrl JavaScript runtime hits; no .url key). Cited as 143. Used in Ch. 6.

## Detection and tradecraft

- <a id="tharros-com"></a>**tharros.com.** [https://tharros.com/the-dangers-of-windows-internetshortcut-url-files/](https://tharros.com/the-dangers-of-windows-internetshortcut-url-files/). (Will Dormann YARA rules: whitespace-tolerant header, wide/UTF-16 variant). Cited as 49. Used in Ch. 2.

- <a id="internalallthethings-iconfile-attacker-share-ntlm-coercion-on-fo"></a>**InternalAllTheThings: IconFile=\\attacker\share NTLM coercion on folder view.** [https://swisskyrepo.github.io/InternalAllTheThings/active-directory/internal-shares/](https://swisskyrepo.github.io/InternalAllTheThings/active-directory/internal-shares/). Cited as 76. Used in Ch. 3.

- <a id="osanda-malith-jayathissa-places-of-interest-in-stealing-netntlm-"></a>**Osanda Malith Jayathissa, "Places of Interest in Stealing NetNTLM Hashes".** [https://osandamalith.com/2017/03/24/places-of-interest-in-stealing-netntlm-hashes/](https://osandamalith.com/2017/03/24/places-of-interest-in-stealing-netntlm-hashes/). (2017 .url URL=file:// primitive). Cited as 118. Used in Ch. 5.

- <a id="penetration-testing-lab-smb-share-scf-file-attacks"></a>**Penetration Testing Lab, "SMB Share – SCF File Attacks".** [https://pentestlab.blog/2017/12/13/smb-share-scf-file-attacks/](https://pentestlab.blog/2017/12/13/smb-share-scf-file-attacks/). (IconFile=\\host\share\pentestlab.ico recipe; Responder capture). Cited as 119. Used in Ch. 5.

- <a id="documentation-of-ntlm-theft-filetype-field-coverage"></a>**documentation of ntlm_theft filetype/field coverage.** [https://www.rootshellsecurity.net/ntlm-hash-disclosure/](https://www.rootshellsecurity.net/ntlm-hash-disclosure/). Cited as 120. Used in Ch. 5.

- <a id="juan-diego-adv170014-scf-hash-theft-disclosure"></a>**Juan Diego, ADV170014 SCF hash-theft disclosure.** [https://www.openwall.com/lists/oss-security/2017/10/24/1](https://www.openwall.com/lists/oss-security/2017/10/24/1). (Microsoft guidance-only response); insert-script.blogspot.com 2018 "Leaking Environment Variables" (%USERNAME% in IconFile). Cited as 121. Used in Ch. 5.

- <a id="windows-search-and-webdav-payloads"></a>**"Windows Search And WebDAV Payloads".** [https://tradecraft.cafe/Windows-Search-And-WebDAV-Payloads/](https://tradecraft.cafe/Windows-Search-And-WebDAV-Payloads/). (verbatim search-ms .url specimen). Cited as 126. Used in Ch. 5.

- <a id="lolbas-ieframe-dll"></a>**LOLBAS Ieframe.dll.** [https://lolbas-project.github.io/lolbas/Libraries/Ieframe/](https://lolbas-project.github.io/lolbas/Libraries/Ieframe/). (OpenURL valid on "Windows 10, Windows 11"); companion entries: /lolbas/Libraries/Url/ and /lolbas/Libraries/Shdocvw/. Cited as 135, 161. Used in Ch. 5, Ch. 6.

- <a id="lolbas-url-dll"></a>**LOLBAS, Url.dll.** [https://lolbas-project.github.io/lolbas/Libraries/Url/](https://lolbas-project.github.io/lolbas/Libraries/Url/). (OpenURL, FileProtocolHandler with caret obfuscation, TelnetProtocolHandler). Cited as 136. Used in Ch. 5.

- <a id="lolbas-shdocvw-dll"></a>**LOLBAS, Shdocvw.dll.** [https://lolbas-project.github.io/lolbas/Libraries/Shdocvw/](https://lolbas-project.github.io/lolbas/Libraries/Shdocvw/). (OpenURL). Cited as 137. Used in Ch. 5.

- <a id="bohops-abusing-exported-functions-and-exposed-dcom-interfaces"></a>**bohops, "Abusing Exported Functions and Exposed DCOM Interfaces".** [https://bohops.com/2018/03/17/abusing-exported-functions-and-exposed-dcom-interfaces-for-pass-thru-command-execution-and-lateral-movement/](https://bohops.com/2018/03/17/abusing-exported-functions-and-exposed-dcom-interfaces-for-pass-thru-command-execution-and-lateral-movement/). (three OpenURL exports; NULL-verb ShellExecute; ShellBrowserWindow DCOM lateral movement). Cited as 139. Used in Ch. 5.

- <a id="sigma-potentially-suspicious-rundll32-activity"></a>**Sigma, "Potentially Suspicious Rundll32 Activity".** [https://detection.fyi/sigmahq/sigma/windows/process_creation/proc_creation_win_rundll32_susp_activity/](https://detection.fyi/sigmahq/sigma/windows/process_creation/proc_creation_win_rundll32_susp_activity/). (url.dll/ieframe.dll/shdocvw.dll OpenURL command lines). Cited as 140. Used in Ch. 5.

## Community reports

- <a id="windows-11-registry-dump"></a>**Windows 11 registry dump.** [https://www.tenforums.com/general-support/193229-windows-file-explorer-wont-execute-url-shortcuts-3.html](https://www.tenforums.com/general-support/193229-windows-file-explorer-wont-execute-url-shortcuts-3.html). (ieframe.dll,OpenURL %l). Cited as 12, 158. Used in Ch. 1, Ch. 6.

- <a id="real-ie8-generated-favorite-with-baseurl-in-default"></a>**real IE8-generated favorite with BASEURL= in [DEFAULT].** [https://ubuntugenius.wordpress.com/2009/12/09/how-to-open-url-internet-explorer-shortcuts-in-ubuntu-using-firefox/](https://ubuntugenius.wordpress.com/2009/12/09/how-to-open-url-internet-explorer-shortcuts-in-ubuntu-using-firefox/). Cited as 26, 63, 147. Used in Ch. 2, Ch. 3, Ch. 6.

- <a id="darkthread-blog-bookmarklet-generator"></a>**darkthread blog: bookmarklet generator.** [https://blog.darkthread.net/blog/ie-bookmarklet/](https://blog.darkthread.net/blog/ie-bookmarklet/). UTF-16LE requirement for CJK; URL/ExtendedURL sync. Cited as 28, 73. Used in Ch. 2, Ch. 3.

- <a id="sap-community-monitoreditem-feedurl-islivepreview-specimen"></a>**SAP Community: [MonitoredItem] FeedUrl/IsLivePreview specimen.** [https://community.sap.com/t5/application-development-discussions/sending-url-as-attachment/m-p/9339383](https://community.sap.com/t5/application-development-discussions/sending-url-as-attachment/m-p/9339383). Cited as 30, 82. Used in Ch. 2, Ch. 3.

- <a id="microsoftdocs-mixed-reality-repo"></a>**MicrosoftDocs/mixed-reality repo.** [https://github.com/MicrosoftDocs/mixed-reality/blob/docs/mixed-reality-docs/mr-dev-docs/distribute/implementing-3d-app-launchers-win32.md](https://github.com/MicrosoftDocs/mixed-reality/blob/docs/mixed-reality-docs/mr-dev-docs/distribute/implementing-3d-app-launchers-win32.md). (sample .URL launcher with {9F4C2855} Prop31/Prop5). Cited as 34. Used in Ch. 2.

- <a id="autoit-forum-full-real-website-specimen"></a>**AutoIt forum: full real .website specimen.** [https://www.autoitscript.com/forum/topic/150926-internet-shortcut-sanitizer-url-and-website-files/](https://www.autoitscript.com/forum/topic/150926-internet-shortcut-sanitizer-url-and-website-files/). (Prop12 variant; Prop*-only keys). Cited as 36, 153. Used in Ch. 2, Ch. 6.

- <a id="spiceworks-community"></a>**Spiceworks community.** [https://community.spiceworks.com/t/microsoft-edge-desktop-shortcut/790553](https://community.spiceworks.com/t/microsoft-edge-desktop-shortcut/790553). (verbatim google.com .website specimen). Cited as 37, 91. Used in Ch. 2, Ch. 4.

- <a id="polish-support-forum-reproducing-the-real-suggested-sites-url"></a>**Polish support forum reproducing the real "Suggested Sites.url".** [https://forum.hotfix.pl/problemy/problem-ze-skrotem-uslugi-sugerowane-witryny-w-ie-t21332.html](https://forum.hotfix.pl/problemy/problem-ze-skrotem-uslugi-sugerowane-witryny-w-ie-t21332.html). FeedUrl + PreviewSize=320x240 + IsLivePreview=true. Cited as 44, 66. Used in Ch. 2, Ch. 3.

- <a id="zdnet-ie9-power-tips-the-secrets-of-pinned-site-shortcuts"></a>**ZDNet, "IE9 power tips: the secrets of pinned site shortcuts".** [https://www.zdnet.com/article/ie9-power-tips-the-secrets-of-pinned-site-shortcuts/](https://www.zdnet.com/article/ie9-power-tips-the-secrets-of-pinned-site-shortcuts/). (.website extension invokes iexplore.exe -w "%l" %*). Cited as 45, 151. Used in Ch. 2, Ch. 6.

- <a id="51cto-blog-browserflags-as-hkcr-registry-value"></a>**51cto blog: BrowserFlags as HKCR registry value.** [https://blog.51cto.com/qicaiiwang/432259](https://blog.51cto.com/qicaiiwang/432259). (registry/file-format confusion). Cited as 50, 146. Used in Ch. 2, Ch. 6.

- <a id="cnblogs-field-status-survey"></a>**cnblogs field-status survey.** [https://www.cnblogs.com/suv789/p/18324691](https://www.cnblogs.com/suv789/p/18324691). Explorer prefers IDList; Comment/Desc not shown on Win10/11; Author/WhatsNew/Roamed legacy; negative IconIndex claim (unverified). Cited as 61, 149. Used in Ch. 3, Ch. 6.

- <a id="kodi-forum-paste-of-epic-games-launcher-url"></a>**Kodi forum paste of Epic Games Launcher .url.** [https://forum.kodi.tv/showthread.php?tid=287826&page=120](https://forum.kodi.tv/showthread.php?tid=287826&page=120). (Prop3=19,0). Cited as 71, 98. Used in Ch. 3, Ch. 4.

- <a id="steam-url-specimens"></a>**Steam .url specimens.** [https://landenlabs.com/cs-urlcleaner/urlcleaner.html](https://landenlabs.com/cs-urlcleaner/urlcleaner.html). (URL=steam://rungameid/<id>, games\<hash>.ico IconFile). Cited as 72. Used in Ch. 3.

- <a id="ars-technica-forum"></a>**Ars Technica forum.** [https://arstechnica.com/civis/threads/whats-the-trick-for-creating-large-icons-for-xp.185275/](https://arstechnica.com/civis/threads/whats-the-trick-for-creating-large-icons-for-xp.185275/). ("I also don't know what the Prop3= key is for"). Cited as 103. Used in Ch. 4.

- <a id="committed-ie9-era-website-specimen"></a>**committed IE9-era .website specimen.** [https://github.com/JONGGON/DeepHumanPrediction](https://github.com/JONGGON/DeepHumanPrediction). (full {A7AF692E} section with Prop2 blob). Cited as 106. Used in Ch. 4.

- <a id="webshortcutsamples-repo"></a>**WebShortcutSamples repo.** [https://github.com/beckus/WebShortcutSamples](https://github.com/beckus/WebShortcutSamples). (hex-verified IE9 Google.website; README: format undocumented). Cited as 107. Used in Ch. 4.

- <a id="greenwolf-ntlm-theft-url-leaks-via-url-and-iconfile-fields-on-fo"></a>**Greenwolf ntlm_theft: .url leaks via URL and ICONFILE fields on folder browse.** [https://github.com/Greenwolf/ntlm_theft](https://github.com/Greenwolf/ntlm_theft). Cited as 113. Used in Ch. 5.

- <a id="trellix-beyond-file-search-a-novel-method"></a>**Trellix, "Beyond File Search: A Novel Method".** [https://www.trellix.com/blogs/research/beyond-file-search-a-novel-method/](https://www.trellix.com/blogs/research/beyond-file-search-a-novel-method/). (search-ms URI anatomy). Cited as 129. Used in Ch. 5.

- <a id="sailay1996-ads-notes"></a>**sailay1996 ADS notes.** [https://github.com/sailay1996/misc-bin/blob/master/ads.md](https://github.com/sailay1996/misc-bin/blob/master/ads.md). (OpenURL on .url content in NTFS alternate data streams). Cited as 138. Used in Ch. 5.

- <a id="sharepoint-diary"></a>**SharePoint Diary.** [https://www.sharepointdiary.com/2020/11/how-to-add-a-link-to-sharepoint-online-document-library.html](https://www.sharepointdiary.com/2020/11/how-to-add-a-link-to-sharepoint-online-document-library.html). (PowerShell $SiteURL variable building a standard URL= .url file). Cited as 142. Used in Ch. 6.

- <a id="winaero"></a>**Winaero.** [https://winaero.com/how-to-use-ie-pinned-sites-on-taskbar-without-disabling-addons/](https://winaero.com/how-to-use-ie-pinned-sites-on-taskbar-without-disabling-addons/). (taskbar pins stored under User Pinned\TaskBar; pinning is external state). Cited as 152. Used in Ch. 6.

- <a id="xp-era-file-association-restore-reg"></a>**XP-era file-association restore .reg.** [https://www.schalley.eu/2009/10/05/restore-windows-file-associations/](https://www.schalley.eu/2009/10/05/restore-windows-file-associations/). (shdocvw.dll,OpenURL %l); corroborated by malware registry analyses. Cited as 156. Used in Ch. 6.

## Weak or contradicted sources

These pages assert format keys that no specimen, SDK header, or parser corroborates. They are cited so the rumor table can point at the claim.

- <a id="filetypedb-com-url-file-format"></a>**filetypedb.com "URL File Format".** [https://filetypedb.com/web/url](https://filetypedb.com/web/url). (weak/uncorroborated aggregator; source of Referer/BrowserFlags/BaseURL rumors). Cited as 42, 69, 144. Used in Ch. 2, Ch. 3, Ch. 6.

- <a id="fileformat-vpscoder-com-url-file-format"></a>**fileformat.vpscoder.com, ".URL File Format".** [https://fileformat.vpscoder.com/task-753](https://fileformat.vpscoder.com/task-753). ShowCommand "1 for normal, 3 for maximized, 7 for minimized" Modified as "inverted FILETIME structure format.". Cited as 58, 169. Used in Ch. 3, Ch. 6.

- <a id="daedalos"></a>**daedalOS.** [https://github.com/DustinBrett/daedalOS/discussions/230](https://github.com/DustinBrett/daedalOS/discussions/230). (web desktop emulator) discussion; only "[InternetShortcut] + BaseURL" hit, not real Windows. Cited as 148. Used in Ch. 6.

## Catalog notes

- <a id="catalog-synthesis"></a>**Catalog synthesis, not an external source.** Research-synthesis insight derived from the Quarkslab, Check Point, Trend Micro, and LOLBAS sources cited below (render-time vs open-time surface split; WebDAV carrier; field-depowering patch pattern). Cited as 108.

## Citation number index

| No. | Work |
|---:|---|
| 1 | [Edward L. Blake, "An Unofficial Guide to the URL File Format"](#edward-l-blake-an-unofficial-guide-to-the-url-file-format) |
| 2 | [rclip-url-file crate docs](#rclip-url-file-crate-docs) |
| 3 | [NSIS wiki, "Creating internet shortcuts"](#nsis-wiki-creating-internet-shortcuts) |
| 4 | [Microsoft Learn, "Internet Shortcuts"](#microsoft-learn-internet-shortcuts) |
| 5 | [Windows 10 SDK ShlGuid.h](#windows-10-sdk-shlguid-h) |
| 6 | [Windows 10 SDK ShlObj.h](#windows-10-sdk-shlobj-h) |
| 7 | [committed .url specimen with [{000214A0-…}] Prop3=19,11](#committed-url-specimen-with-000214a0-prop3-19-11) |
| 8 | [Microsoft Learn, sample .URL launcher shortcut](#microsoft-learn-sample-url-launcher-shortcut) |
| 9 | [NCC Group, "The Case of Missing File Extensions"](#ncc-group-the-case-of-missing-file-extensions) |
| 10 | [registry dump of HKCR\InternetShortcut](#registry-dump-of-hkcr-internetshortcut) |
| 11 | [Quarkslab, CVE-2016-3353 analysis](#quarkslab-cve-2016-3353-analysis) |
| 12 | [Windows 11 registry dump](#windows-11-registry-dump) |
| 13 | [Microsoft Q&A 2150484](#microsoft-q-a-2150484) |
| 14 | [Check Point Research, "Blind Eagle"](#check-point-research-blind-eagle) |
| 15 | [Wine ieframe intshcut.c](#wine-ieframe-intshcut-c) |
| 16 | [Wine ieframe test fixture](#wine-ieframe-test-fixture) |
| 17 | [NSIS wiki, "Creating internet shortcuts"](#nsis-wiki-creating-internet-shortcuts) |
| 18 | [Edward L. Blake, "An Unofficial Guide to the URL File Format"](#edward-l-blake-an-unofficial-guide-to-the-url-file-format) |
| 19 | [Microsoft Learn, "Internet Shortcuts"](#microsoft-learn-internet-shortcuts) |
| 20 | [Windows 10 SDK ShlObj.h](#windows-10-sdk-shlobj-h) |
| 21 | [Windows 10 SDK ShlGuid.h](#windows-10-sdk-shlguid-h) |
| 22 | [Microsoft Learn, sample .URL launcher shortcut](#microsoft-learn-sample-url-launcher-shortcut) |
| 23 | [Microsoft Q&A 2150484](#microsoft-q-a-2150484) |
| 24 | [Check Point Research, "Blind Eagle"](#check-point-research-blind-eagle) |
| 25 | [tatsu-syo.info](#tatsu-syo-info) |
| 26 | [real IE8-generated favorite with BASEURL= in [DEFAULT]](#real-ie8-generated-favorite-with-baseurl-in-default) |
| 27 | [IE bookmarklet .url gist](#ie-bookmarklet-url-gist) |
| 28 | [darkthread blog: bookmarklet generator](#darkthread-blog-bookmarklet-generator) |
| 29 | [Hybrid Analysis sandbox](#hybrid-analysis-sandbox) |
| 30 | [SAP Community: [MonitoredItem] FeedUrl/IsLivePreview specimen](#sap-community-monitoreditem-feedurl-islivepreview-specimen) |
| 31 | [Quarkslab, CVE-2016-3353 analysis](#quarkslab-cve-2016-3353-analysis) |
| 32 | [Microsoft Learn, System.AppUserModel.ID](#microsoft-learn-system-appusermodel-id) |
| 33 | [Microsoft Learn, IE11 deployment guide](#microsoft-learn-ie11-deployment-guide) |
| 34 | [MicrosoftDocs/mixed-reality repo](#microsoftdocs-mixed-reality-repo) |
| 35 | [Check Point Research, CVE-2025-24054](#check-point-research-cve-2025-24054) |
| 36 | [AutoIt forum: full real .website specimen](#autoit-forum-full-real-website-specimen) |
| 37 | [Spiceworks community](#spiceworks-community) |
| 38 | [Microsoft KB](#microsoft-kb) |
| 39 | [metabpa browser-bookmarks project](#metabpa-browser-bookmarks-project) |
| 40 | [Microsoft Q&A](#microsoft-q-a) |
| 41 | [voidtools Everything docs](#voidtools-everything-docs) |
| 42 | [filetypedb.com "URL File Format"](#filetypedb-com-url-file-format) |
| 43 | [ExifTool test-suite .url specimen with all metadata keys populated](#exiftool-test-suite-url-specimen-with-all-metadata-keys-populate) |
| 44 | [Polish support forum reproducing the real "Suggested Sites.url"](#polish-support-forum-reproducing-the-real-suggested-sites-url) |
| 45 | [ZDNet, "IE9 power tips: the secrets of pinned site shortcuts"](#zdnet-ie9-power-tips-the-secrets-of-pinned-site-shortcuts) |
| 46 | [rclip-url-file crate docs](#rclip-url-file-crate-docs) |
| 47 | [rclip-url-file crate docs](#rclip-url-file-crate-docs) |
| 48 | [Stack Overflow specimen](#stack-overflow-specimen) |
| 49 | [tharros.com](#tharros-com) |
| 50 | [51cto blog: BrowserFlags as HKCR registry value](#51cto-blog-browserflags-as-hkcr-registry-value) |
| 51 | [CVE-2024-43451 PoC .url gist](#cve-2024-43451-poc-url-gist) |
| 52 | [Microsoft Learn, "Internet Shortcuts"](#microsoft-learn-internet-shortcuts) |
| 53 | [Windows 10 SDK ShlObj.h](#windows-10-sdk-shlobj-h) |
| 54 | [NSIS wiki, "Creating internet shortcuts"](#nsis-wiki-creating-internet-shortcuts) |
| 55 | [Wine ieframe intshcut.c](#wine-ieframe-intshcut-c) |
| 56 | [Edward L. Blake, "An Unofficial Guide to the URL File Format"](#edward-l-blake-an-unofficial-guide-to-the-url-file-format) |
| 57 | [The Broken Event Blog](#the-broken-event-blog) |
| 58 | [fileformat.vpscoder.com, ".URL File Format"](#fileformat-vpscoder-com-url-file-format) |
| 59 | [rclip-url-file crate docs](#rclip-url-file-crate-docs) |
| 60 | [rclip-url-file crate docs](#rclip-url-file-crate-docs) |
| 61 | [cnblogs field-status survey](#cnblogs-field-status-survey) |
| 62 | [ExifTool test-suite .url specimen with all metadata keys populated](#exiftool-test-suite-url-specimen-with-all-metadata-keys-populate) |
| 63 | [real IE8-generated favorite with BASEURL= in [DEFAULT]](#real-ie8-generated-favorite-with-baseurl-in-default) |
| 64 | [IE bookmarklet .url gist](#ie-bookmarklet-url-gist) |
| 65 | [Hybrid Analysis sandbox](#hybrid-analysis-sandbox) |
| 66 | [Polish support forum reproducing the real "Suggested Sites.url"](#polish-support-forum-reproducing-the-real-suggested-sites-url) |
| 67 | [Wine ieframe test fixture](#wine-ieframe-test-fixture) |
| 68 | [ctrl.blog, "What is the best file format for web shortcuts"](#ctrl-blog-what-is-the-best-file-format-for-web-shortcuts) |
| 69 | [filetypedb.com "URL File Format"](#filetypedb-com-url-file-format) |
| 70 | [Check Point Research, CVE-2025-24054](#check-point-research-cve-2025-24054) |
| 71 | [Kodi forum paste of Epic Games Launcher .url](#kodi-forum-paste-of-epic-games-launcher-url) |
| 72 | [Steam .url specimens](#steam-url-specimens) |
| 73 | [darkthread blog: bookmarklet generator](#darkthread-blog-bookmarklet-generator) |
| 74 | [Blake, 2nd ed](#blake-2nd-ed) |
| 75 | [CVE-2024-43451 PoC .url gist](#cve-2024-43451-poc-url-gist) |
| 76 | [InternalAllTheThings: IconFile=\\attacker\share NTLM coercion on folder view](#internalallthethings-iconfile-attacker-share-ntlm-coercion-on-fo) |
| 77 | [committed .url specimen with [{000214A0-…}] Prop3=19,11](#committed-url-specimen-with-000214a0-prop3-19-11) |
| 78 | [Stack Overflow specimen](#stack-overflow-specimen) |
| 79 | [registry dump of HKCR\InternetShortcut](#registry-dump-of-hkcr-internetshortcut) |
| 80 | [metabpa browser-bookmarks project](#metabpa-browser-bookmarks-project) |
| 81 | [Microsoft KB](#microsoft-kb) |
| 82 | [SAP Community: [MonitoredItem] FeedUrl/IsLivePreview specimen](#sap-community-monitoreditem-feedurl-islivepreview-specimen) |
| 83 | [Committed 2022 .website specimen](#committed-2022-website-specimen) |
| 84 | [NSIS wiki, "Creating internet shortcuts"](#nsis-wiki-creating-internet-shortcuts) |
| 85 | [Microsoft Learn, "Internet Shortcuts"](#microsoft-learn-internet-shortcuts) |
| 86 | [Windows 10 SDK ShlObj.h](#windows-10-sdk-shlobj-h) |
| 87 | [Microsoft Learn, System.AppUserModel.ID](#microsoft-learn-system-appusermodel-id) |
| 88 | [Microsoft Learn, sample .URL launcher shortcut](#microsoft-learn-sample-url-launcher-shortcut) |
| 89 | [Wine ieframe test fixture](#wine-ieframe-test-fixture) |
| 90 | [Check Point Research, CVE-2025-24054](#check-point-research-cve-2025-24054) |
| 91 | [Spiceworks community](#spiceworks-community) |
| 92 | [metabpa browser-bookmarks project](#metabpa-browser-bookmarks-project) |
| 93 | [Microsoft KB](#microsoft-kb) |
| 94 | [Microsoft Q&A](#microsoft-q-a) |
| 95 | [voidtools Everything docs](#voidtools-everything-docs) |
| 96 | [Windows 10 SDK ShlGuid.h](#windows-10-sdk-shlguid-h) |
| 97 | [Steam shortcut gist](#steam-shortcut-gist) |
| 98 | [Kodi forum paste of Epic Games Launcher .url](#kodi-forum-paste-of-epic-games-launcher-url) |
| 99 | [Quarkslab, CVE-2016-3353 analysis](#quarkslab-cve-2016-3353-analysis) |
| 100 | [Check Point Research, "Blind Eagle"](#check-point-research-blind-eagle) |
| 101 | [committed .url specimen](#committed-url-specimen) |
| 102 | [IE bookmarklet .url gist](#ie-bookmarklet-url-gist) |
| 103 | [Ars Technica forum](#ars-technica-forum) |
| 104 | [Stack Overflow specimen](#stack-overflow-specimen) |
| 105 | [tatsu-syo.info](#tatsu-syo-info) |
| 106 | [committed IE9-era .website specimen](#committed-ie9-era-website-specimen) |
| 107 | [WebShortcutSamples repo](#webshortcutsamples-repo) |
| 108 | [Catalog synthesis, not an external source](#catalog-synthesis) |
| 109 | [Microsoft Q&A 2150484](#microsoft-q-a-2150484) |
| 110 | [Quarkslab, CVE-2016-3353 analysis](#quarkslab-cve-2016-3353-analysis) |
| 111 | [Check Point Research, "Stealth Falcon Zero-Day"](#check-point-research-stealth-falcon-zero-day) |
| 112 | [Check Point Research, CVE-2025-24054](#check-point-research-cve-2025-24054) |
| 113 | [Greenwolf ntlm_theft: .url leaks via URL and ICONFILE fields on folder browse](#greenwolf-ntlm-theft-url-leaks-via-url-and-iconfile-fields-on-fo) |
| 114 | [Microsoft Learn, "Internet Shortcuts"](#microsoft-learn-internet-shortcuts) |
| 115 | [Edward L. Blake, "An Unofficial Guide to the URL File Format"](#edward-l-blake-an-unofficial-guide-to-the-url-file-format) |
| 116 | [Check Point Research, "Blind Eagle"](#check-point-research-blind-eagle) |
| 117 | [CVE-2024-43451 PoC .url gist](#cve-2024-43451-poc-url-gist) |
| 118 | [Osanda Malith Jayathissa, "Places of Interest in Stealing NetNTLM Hashes"](#osanda-malith-jayathissa-places-of-interest-in-stealing-netntlm-) |
| 119 | [Penetration Testing Lab, "SMB Share – SCF File Attacks"](#penetration-testing-lab-smb-share-scf-file-attacks) |
| 120 | [documentation of ntlm_theft filetype/field coverage](#documentation-of-ntlm-theft-filetype-field-coverage) |
| 121 | [Juan Diego, ADV170014 SCF hash-theft disclosure](#juan-diego-adv170014-scf-hash-theft-disclosure) |
| 122 | [Kaspersky Securelist, "NTLM abuse in 2025"](#kaspersky-securelist-ntlm-abuse-in-2025) |
| 123 | [0patch, "Micropatches Released for WebDAV Remote Code Execution (CVE-2025-33053)"](#0patch-micropatches-released-for-webdav-remote-code-execution-cv) |
| 124 | [Rapid7, analysis of exposed attacker WebDAV test server](#rapid7-analysis-of-exposed-attacker-webdav-test-server) |
| 125 | [Google Project Zero #693](#google-project-zero-693) |
| 126 | ["Windows Search And WebDAV Payloads"](#windows-search-and-webdav-payloads) |
| 127 | [itresit labs](#itresit-labs) |
| 128 | [Trend Micro ZDI, "CVE-2024-21412: Water Hydra Targets Traders"](#trend-micro-zdi-cve-2024-21412-water-hydra-targets-traders) |
| 129 | [Trellix, "Beyond File Search: A Novel Method"](#trellix-beyond-file-search-a-novel-method) |
| 130 | [ZDI-16-506 advisory](#zdi-16-506-advisory) |
| 131 | [mirror of M01N Team analysis of CVE-2023-36025](#mirror-of-m01n-team-analysis-of-cve-2023-36025) |
| 132 | [Trend Micro, "CVE-2023-36025 Exploited for Defense Evasion in Phemedrone Stealer...](#trend-micro-cve-2023-36025-exploited-for-defense-evasion-in-phem) |
| 133 | [Trend Micro, "DarkGate Operators Exploit Microsoft Windows SmartScreen Bypass"](#trend-micro-darkgate-operators-exploit-microsoft-windows-smartsc) |
| 134 | [Microsoft Security Update Guide, CVE-2024-21412](#microsoft-security-update-guide-cve-2024-21412) |
| 135 | [LOLBAS Ieframe.dll](#lolbas-ieframe-dll) |
| 136 | [LOLBAS, Url.dll](#lolbas-url-dll) |
| 137 | [LOLBAS, Shdocvw.dll](#lolbas-shdocvw-dll) |
| 138 | [sailay1996 ADS notes](#sailay1996-ads-notes) |
| 139 | [bohops, "Abusing Exported Functions and Exposed DCOM Interfaces"](#bohops-abusing-exported-functions-and-exposed-dcom-interfaces) |
| 140 | [Sigma, "Potentially Suspicious Rundll32 Activity"](#sigma-potentially-suspicious-rundll32-activity) |
| 141 | [Microsoft, "Internet Explorer 11 desktop app retirement FAQ"](#microsoft-internet-explorer-11-desktop-app-retirement-faq) |
| 142 | [SharePoint Diary](#sharepoint-diary) |
| 143 | [Hybrid Analysis sandbox memory scan](#hybrid-analysis-sandbox-memory-scan) |
| 144 | [filetypedb.com "URL File Format"](#filetypedb-com-url-file-format) |
| 145 | [Microsoft Learn, "Internet Shortcuts"](#microsoft-learn-internet-shortcuts) |
| 146 | [51cto blog: BrowserFlags as HKCR registry value](#51cto-blog-browserflags-as-hkcr-registry-value) |
| 147 | [real IE8-generated favorite with BASEURL= in [DEFAULT]](#real-ie8-generated-favorite-with-baseurl-in-default) |
| 148 | [daedalOS](#daedalos) |
| 149 | [cnblogs field-status survey](#cnblogs-field-status-survey) |
| 150 | [ss64: negative IconIndex = resource ID rule stated for Desktop.ini, not .url](#ss64-negative-iconindex-resource-id-rule-stated-for-desktop-ini-) |
| 151 | [ZDNet, "IE9 power tips: the secrets of pinned site shortcuts"](#zdnet-ie9-power-tips-the-secrets-of-pinned-site-shortcuts) |
| 152 | [Winaero](#winaero) |
| 153 | [AutoIt forum: full real .website specimen](#autoit-forum-full-real-website-specimen) |
| 154 | [Check Point Research, CVE-2025-24054](#check-point-research-cve-2025-24054) |
| 155 | [Hybrid Analysis sandbox](#hybrid-analysis-sandbox) |
| 156 | [XP-era file-association restore .reg](#xp-era-file-association-restore-reg) |
| 157 | [Quarkslab, CVE-2016-3353 analysis](#quarkslab-cve-2016-3353-analysis) |
| 158 | [Windows 11 registry dump](#windows-11-registry-dump) |
| 159 | [metabpa browser-bookmarks project](#metabpa-browser-bookmarks-project) |
| 160 | [Microsoft, "Internet Explorer 11 desktop app retirement FAQ"](#microsoft-internet-explorer-11-desktop-app-retirement-faq) |
| 161 | [LOLBAS Ieframe.dll](#lolbas-ieframe-dll) |
| 162 | [0patch, "Micropatches Released for WebDAV Remote Code Execution (CVE-2025-33053)"](#0patch-micropatches-released-for-webdav-remote-code-execution-cv) |
| 163 | [Rapid7, analysis of exposed attacker WebDAV test server](#rapid7-analysis-of-exposed-attacker-webdav-test-server) |
| 164 | [metabpa browser-bookmarks project](#metabpa-browser-bookmarks-project) |
| 165 | [Edward L. Blake, "An Unofficial Guide to the URL File Format"](#edward-l-blake-an-unofficial-guide-to-the-url-file-format) |
| 166 | [rclip-url-file crate docs](#rclip-url-file-crate-docs) |
| 167 | [Windows 10 SDK ShlObj.h](#windows-10-sdk-shlobj-h) |
| 168 | [Wine source tree](#wine-source-tree) |
| 169 | [fileformat.vpscoder.com, ".URL File Format"](#fileformat-vpscoder-com-url-file-format) |
| 170 | [Microsoft Q&A 2150484](#microsoft-q-a-2150484) |
| 171 | [tatsu-syo.info](#tatsu-syo-info) |

## Sources used only in abuse cases

These footnotes live in [Abuse cases](docs/06-abuse-cases.md) and are not part of the global numbering above.

- **Trend Micro, "CVE-2024-21412 Facts and Fixes".** [https://www.trendmicro.com/en_us/research/24/b/cve-2024-21412-facts-and-fixes.html](https://www.trendmicro.com/en_us/research/24/b/cve-2024-21412-facts-and-fixes.html). Abuse-cases footnote 10.

- **Penetration Testing Lab, "Initial Access – Search-ms URI Handler".** [https://pentestlab.blog/2024/01/02/initial-access-search-ms-uri-handler/](https://pentestlab.blog/2024/01/02/initial-access-search-ms-uri-handler/). Abuse-cases footnote 15.

- **Forcepoint X-Labs.** [https://www.forcepoint.com/blog/x-labs/asyncrat-python-trycloudflare-malware](https://www.forcepoint.com/blog/x-labs/asyncrat-python-trycloudflare-malware). Abuse-cases footnote 16.

- **hexacorn, "Running programs via Proxy & jumping on a EDR-bypass trampoline".** [https://www.hexacorn.com/blog/2017/05/01/running-programs-via-proxy-jumping-on-a-edr-bypass-trampoline/](https://www.hexacorn.com/blog/2017/05/01/running-programs-via-proxy-jumping-on-a-edr-bypass-trampoline/). Abuse-cases footnote 25.

- **hexacorn Part 5.** [https://www.hexacorn.com/blog/2018/03/15/running-programs-via-proxy-jumping-on-a-edr-bypass-trampoline-part-5/](https://www.hexacorn.com/blog/2018/03/15/running-programs-via-proxy-jumping-on-a-edr-bypass-trampoline-part-5/). Abuse-cases footnote 26.

- **Microsoft Security Bulletin MS16-104.** [https://learn.microsoft.com/en-us/security-updates/securitybulletins/2016/ms16-104](https://learn.microsoft.com/en-us/security-updates/securitybulletins/2016/ms16-104). Abuse-cases footnote 42.

- **InQuest/OPSWAT, "Shortcut to Malice: URL Files".** [https://inquest.net/blog/shortcut-to-malice-url-files/](https://inquest.net/blog/shortcut-to-malice-url-files/). Abuse-cases footnote 43.

- **Sigma, CVE-2025-33053 exploit detection.** [https://detection.fyi/sigmahq/sigma/emerging-threats/2025/exploits/cve-2025-33053/](https://detection.fyi/sigmahq/sigma/emerging-threats/2025/exploits/cve-2025-33053/). Abuse-cases footnote 49.

- **HackTricks WebDAV page.** [https://hacktricks.wiki](https://hacktricks.wiki). Abuse-cases footnote 50.
