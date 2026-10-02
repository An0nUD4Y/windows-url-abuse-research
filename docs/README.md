# Chapters

Read in this order. Each chapter ends with the footnotes it uses, so a GitHub page stands on its own. The shared numbering is explained in the [bibliography](../references.md).

| | Chapter | What you get |
|---|---|---|
| 1 | [Introduction, scope, and method](01-introduction.md) | What a `.url` file is, which component parses it, render-time versus open-time, and the four confidence labels |
| 2 | [Sections](02-sections.md) | Every section name, the `.A` / `.W` shadows, frameset sections, `[Bookmarklet]`, `[MonitoredItem]`, GUID property sections, and the `.website` container |
| 3 | [Keys](03-keys.md) | Every `name=value` key, the 2083 / 5119 limits, the `Modified` FILETIME decode, and the `HotKey` encoding |
| 4 | [Property bags](04-property-bags.md) | The `Prop<N>=<vt>,<value>` grammar, VT tags, and the PID_IS_* / PID_INTSITE_* tables, including PROPIDs that are observed and undocumented |
| 5 | [Security behaviors](05-security-behaviors.md) | Field risk matrix, NTLM leaks, `WorkingDirectory` hijack, MotW and SmartScreen, OpenURL exports, and which patch closed which field |
| — | [Abuse cases](06-abuse-cases.md) | Campaign walkthroughs, ATT&CK mapping, and a per-field status table. Footnote numbers in this file start at 1 and apply only here |
| 6 | [Rumors, versions, and gaps](07-rumors-versions-gaps.md) | Keys that were claimed and not corroborated, behavior by Windows generation, and questions the catalog leaves open |

## Confidence labels

| Label | Meaning |
|---|---|
| **[Documented]** | Appears in Microsoft documentation or Windows SDK headers |
| **[Observed]** | Seen in a real file written by Microsoft software or in an in-the-wild specimen, and that specimen is cited |
| **[RE'd]** | Established from binary analysis or from Wine / ReactOS source, and that code is cited |
| **[Rumor]** | Claimed, and not corroborated. These rows live only in chapter 6 |

A key with no specimen, no SDK property ID, and no parser is a rumor. It is not recorded as a deprecated feature.

## Adding a claim

1. Put the claim in the chapter that owns it. An unverified key goes in chapter 6, not into the master tables in chapters 2–4.
2. Give it one confidence label.
3. Cite it with the next free global number (`[^172]:` and up) at the bottom of that chapter. Abuse cases is the exception: add the footnote to that file's own list.
4. Add the work to [references.md](../references.md) under the matching source group, and add the new number to the citation index so it points at that work.

[Specimens](../specimens/README.md) · [Bibliography](../references.md) · [Catalog home](../README.md)
