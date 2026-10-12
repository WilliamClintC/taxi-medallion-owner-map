# Figure 116: where the owners, lenders and agents in our data filed from (interactive map)

## Goal

- Ed, 2026-10-12: "Do all taxi drivers/companies live in New York and New Jersey? I need to know their locations in order to request bankruptcy data from the appropriate court."
- Map every owner, lender and TLC managing agent we hold an address for, and give each one's federal (bankruptcy) district.

## Output

- `116_owner_lender_locations_map.html`: published on GitHub Pages (https://williamclintc.github.io/taxi-medallion-owner-map/), names and addresses shown. Leaflet 1.9.4 and markercluster 1.5.3 are inlined. Data is in the sibling `116_owner_lender_locations_map_data.js` (7.86 MB since pass 6; county shapes and court names added in pass 5).
- Six groups, colour by family (owners blue, lenders orange, agents teal) and shape by group, used the same way on the map, in the legend, in popups, chips and the counts table (pass 5; pass 1 to 4 used six Okabe-Ito hues):

| Group | Shape | Colour | Points | Names mapped |
|---|---|---|---|---|
| Individual owners | circle | #7fb3ef blue (mid) | 17,027 | 4,597 |
| Companies (LLC) | square | #c4dcf8 blue (light) | 2,498 | 1,615 |
| Companies (other: INC, CORP, etc.) | diamond | #2f7fe0 blue (dark) | 14,278 | 4,120 |
| Lenders | triangle | #d95926 orange | 1,339 | 579 |
| Lender-held series LLCs (730 series + 2 DePalma acquisition vehicles) | down triangle | #f4a27c orange (light) | 732 | 732 |
| TLC managing agents | star (light halo, larger) | #199e70 teal | 95 | 82 |
| TLC drivers | no map element | | 0 | 0 |
| Crash drivers | no map element | | 0 | 0 |

- Colours (pass 5, dark page): base hues #3987e5, #d95926, #199e70 pass the dataviz validator (dark, surface #1c1d20, all pairs: CVD worst 9.4 deutan, normal 20.9, all at least 3:1). Owner shades #2f7fe0 / #7fb3ef / #c4dcf8 and lender shades #d95926 / #f4a27c pass the ordinal check (adjacent lightness gap at least 0.06, at least 4.2:1 on the surface). Shape carries the group within a family.
- Markers are canvas paths (no DOM per point). Pass 3: 39,610 points loaded in about 5 s headless (pass 4: 35,575 points); a layer toggle takes 0.1 to 0.3 s.
- Cluster bubbles: one cluster layer for all groups; each bubble is a donut whose ring shows the group mix.
- Hover a bubble: per-group counts with each group's shape (readable without colour).
- Cluster radius shrinks with zoom (44 px, 20 px from zoom 15, 6 px from zoom 17); same-address stacks still cluster and spiderfy.
- Toggles: each group (pass 5: owners on by default, scoping the court bars and the map); owner points at an address spelling found only on NYS DOS liens (9,938 since pass 6; spelling on no Lexis record of the owner); owner points at a managing-agent office (6,482); district lines. Pass 4 removed the All sources / Lexis-backed only switch (one basis).
- Popups tag: "At a managing-agent office", "Also a TLC managing agent" (11 lenders), "Medallion-number ACQUISITION LLC", and the medallion-test result (lender points; owner points with DOS rows show "lien names this owner's medallion" and the lien's Lexis lender).
- Pass 3 (2026-10-12) screens: `116_desktop_1440_lic16/17/18.png` (Woodside at street zoom), `_lic17_grey.png` (greyscale), `_cluster_tooltip.png`, `_agent_star.png`, `_group_rules.png`; pass-2 renders moved to `_screens/pass2/`.
- `_screens/`: 1440 and 390 px (full page, viewport, counts both bases, layers, drivers, notes, cluster view, popups) and one zoomed crop per shape (`116_desktop_1440_shape_*.png`). No console errors; no sideways scroll at either width. Pass-1 screenshots moved to `_screens/pass1/`.
- Pass 4 screens: `_screens/116_pass4_*.png`. Pass 6: `_screens/pass6/`.

## Pass 2 (2026-10-12): what changed

- **Owner groups** (rules in `_owner_loc_common.owner_group`, applied in `load_entities`; `name_match.py` not edited):

| Rule | Rows | From to |
|---|---|---|
| Individuals list, company word (LIMITED, INCORPORATED, TRANS, TRANSPORTATION, MOTOR, SYSTEM, ASSETS, dotted L.L.C., truncated LL) | 12 | individual to llc 3 (MOJO WORKING SYSTEM,L.L.C.; JAGGI & YASH TRANSPORTATION LL; RCMK, L. L. C.), to company_other 9 |
| LOMTO FEDERAL CREDIT UNION (bank / credit union / servicer name) | 1 | individual to out of owner groups |
| LATKA, SIMKA, WHEELER, BANTA, REIGER, GRAVAS, IGNATOWSKI + LLC | 735 (1,455 medallions) | llc to out of owner groups (lender-held series LLCs) |
| LIMOUSINE COMMISS, AUCTION WINNER | 2 (202 medallions) | company_other to out of owner groups |
| (IN LIVING TRUST) | 1 | company_other to individual |

  - RCMK, L. L. C. was not in the audit's 12; the audit's own regex catches it.
  - None of the 738 excluded rows had a mapped point, so owner points and names mapped are unchanged.
- **Lenders** (261b):
  - Medallion test per DOS v1 lien against Lexis `ucc_records.csv`: lien on Lexis with a passing row (attribution not no-medallion, or MEDALLION in the collateral) kept; lien not on Lexis kept when the family's Lexis medallion share is 0.5 or more; never by keyword.
  - Points by test: lien 1,154, family 136, Lexis non-medallion but family share 0.5 or more ("c2") 47 kept as an open question. 1,673 lien-party rows dropped.
  - Only secured_party_type Organization or Individual (2 rows dropped). 26 liens name only a filing agent (lender unknown).
  - Label clean-up, only where the canonical label still carries the junk: trailing underscores (Progressive Credit Union____ to Progressive CU), bracketed share (Flushing Savings Bank Fsb [94 31579%] to Flushing Savings Bank), "As Custodian For" (3 labels to Israel Discount Bank Of New York As Custodian). Pierre Funding stays Pierre Funding.
  - Fragment labels take the family with the most records at the same address: M to MFIN, N and We to NAA Funding, R to Rapid Funding, V to Capital One.
- **Lender-held series LLCs**: own group (`lender_spv`). 732 kept = 730 series LLCs that hold a medallion on the owner list + DePalma Acquisition I and II (no medallion; both at 127 W 26th St 10001). 170 series LLCs with no medallion dropped. Septuagint Solutions is an agent only.
  - 732 equals the 732 owner-side twins by coincidence: the twins are 732 of the 735 excluded owner rows (no twin: LATKA TRAXI 254 LLC, SIMKA TACI 273 LLC, SIMKA YTAXI 230 LLC).
- **Agents**: 108 agent rows collapsed to 82 names (name_match token set); addresses collapsed by the loose key. Done in 261f so the 5 GB TLC read is not repeated.
- **Managing-agent offices**: `loose_key` removes C/O, house-number hyphens (54-11 = 5411) and ordinals (44TH = 44). 6,815 owner points sit at an agent office. 2,628 of 10,332 owners have an agent office as their latest address; 949 have no other mapped address. District columns now use each owner's latest NON-agent address when one exists.
- **Entity counts**: `distinct_names` keys on the name_match token set after removing a trailing `__2_3` artifact.

### Group counts, before and after (all sources)

| Group | Names on list, pass 1 | Names on list, pass 2 | Names mapped, pass 1 | Names mapped, pass 2 | Points, pass 1 | Points, pass 2 |
|---|---|---|---|---|---|---|
| Individual owners | 5,129 | 5,117 | 4,602 | 4,597 | 23,298 | 23,286 |
| Companies (LLC) | 4,684 | 3,952 | 1,614 | 1,615 | 2,616 | 2,617 |
| Companies (other) | 5,152 | 5,158 | 4,116 | 4,120 | 15,568 | 15,579 |
| **All owners** | 14,965 | 14,227 | 10,332 | 10,332 | 41,482 | 41,482 |
| Lenders | n/a | n/a | 1,889 (incl. 903 series LLCs) | 578 | 2,705 mapped (1,802 L, 903 S) | 1,337 |
| Lender-held series LLCs (730 series + 2 DePalma) | in Lenders | n/a | in Lenders | 732 | in Lenders | 732 |
| TLC managing agents | n/a | n/a | 108 | 82 | 154 | 95 |
| All points | | | | | 44,341 | 43,646 |

- Owner points unchanged in total (41,482); 12 moved from individual to the company groups.

## Counts (stage 261f, re-run 2026-10-12 pass 6, own-medallion rule with the lien-number fix)

District columns count each name once, at its latest mapped address that is not a managing-agent office (else its latest address). "Any address outside NY/NJ" counts names with at least one mapped address outside SDNY, EDNY, NDNY, WDNY and D.N.J.

All sources (the only basis since pass 4). Pass-4 values in the pass-6 section below.

| Group | On list | Mapped | Addresses | SDNY | EDNY | NDNY | WDNY | D.N.J. | Other | Any address outside NY/NJ |
|---|---|---|---|---|---|---|---|---|---|---|
| Individual owners | 5,117 | 4,597 | 13,688 | 485 | 3,546 | 6 | 4 | 342 | 214 | 314 |
| Companies (LLC) | 3,952 | 1,615 | 1,492 | 333 | 1,200 | 1 | 0 | 73 | 8 | 10 |
| Companies (other) | 5,158 | 4,120 | 5,396 | 968 | 2,954 | 1 | 2 | 141 | 54 | 67 |
| **All owners** | 14,227 | 10,332 | 19,318 | 1,786 | 7,700 | 8 | 6 | 556 | 276 | 391 |
| Lenders | n/a | 579 | 1,085 | 161 | 242 | 12 | 3 | 50 | 111 | 122 |
| Lender-held series LLCs (730 series + 2 DePalma) | n/a | 732 | 17 | 478 | 254 | 0 | 0 | 0 | 0 | 0 |
| TLC managing agents | n/a | 82 | 78 | 17 | 65 | 0 | 0 | 0 | 0 | 0 |
| TLC drivers | 56,648 | 0 | 0 | | | | | | | |
| Crash drivers | n/a | 0 | 0 | | | | | | | |

- **Owners: 10,056 of the 10,332 mapped (97.3%) have their latest non-agent address in NY or NJ** (pass 4 the same; pass 3: 10,060, 97.4%; pass 1, latest address of any kind: 10,111, 97.9%).
  - The other 276: S.D. Fla. 101, M.D. Fla. 54, D. Conn. 18, M.D. Pa. 11, E.D. Pa. 11, N.D. Ga. 9, C.D. Cal. 8, W.D. Wash. 5, and 59 more in 36 other districts.
  - 5 of the 276 rest only on DOS addresses (pass 3: 6 of 272).
- DOS v1 and v2 owner rows now count only when the lien, on Lexis by filing number, names the owner's own medallion. 9,938 of 33,803 owner points (29.4%) sit at a DOS-only address spelling.
- Managing-agent office: 6,482 owner points; latest address for 2,671 owners, 1,035 with no other mapped address.
- LLC row includes 268 medallion-number ACQUISITION LLCs (9 mapped, 10 points).
- 238 list rows repeat a name by token set (distinct: individuals 4,986, LLC 3,921, other 5,082).

## Drivers

- **TLC drivers: 56,648 hack licences, 51,508 named, 0 with an address (no source holds one).** No map element.
- **Crash drivers** (`medallion_crash_individuals_panel.csv`, roles Driver of a Motor Vehicle in Transport or Driver -- Missing Information), 2018 to 2024, N = 18,374 driver rows on 9,208 medallions; no name or hack-licence link:

| Licence state | Driver rows | Share |
|---|---|---|
| NY | 17,269 | 94.0% |
| NJ | 788 | 4.3% |
| PA | 108 | 0.6% |
| CT | 27 | 0.1% |
| Other | 80 | 0.4% |
| Blank | 102 | 0.6% |
| **All** | **18,374** | |

- Other: NC 10, OT 9, FL 8, UN 8, MD 6, CA 5, WA 4, DE 3, and others.
- **Crash-county layer NOT built.** The NYS Case Information file has no case key: 18 columns, none an identifier (`data/0_raw/nys_dmv_crashes/README.md`, "Case Information does not join to anything"). A year + county + date join would manufacture matches. So there is no county count, no county top 10, and no choropleth.

## Open group questions (for William)

1. **c2 lender points (47).** Lexis holds the lien and lists it as non-medallion, but the lender family is mostly medallion loans (Capital One, MFIN, Popular and others). The audit rule drops them; kept for now, tagged in the popup.
2. **Lender name folds (51 labels, about 54 points)** belong in `build_lender_name_canonical_map.py`: Mega Fundig, Medallionf Udning, North Forkk Bank, Peal G, Banco Poular, Signataure Bank, and others. Not done here, since that map feeds figures 82 to 115. Until then `assert_no_unmapped_variants` cannot be added to 261b (it would fail).
3. **"Ogs"** (raw THE OGS CORPORATION, 287 10th Ave, OSG Corp's address): 1 lien, now dropped by the medallion test (fails the family test), so not on the map. Folded into OSG Corp, it would come back as an OSG Corp point. Fold? Not a clear typo.
4. **268 medallion-number ACQUISITION LLCs** stay in LLC (footnoted). The 9 mapped sit at 225 Crossways Park Dr 11797 or 5411 Queens Blvd 11377. Who controls them is not established.
5. **21 individual owners match a lender family** (exact or set; audit count, not recomputed): seller financing or namesakes (SINGH, PIERRE). Add a dual-role tag?
6. **Series LLC count**: the owner-side regex takes 735 rows (incl. 5 typo variants); the lender side's name_match pairs 730 DOS corporations with 732 owner rows. The lender_spv group's 732 is 730 series + 2 DePalma, not the 732 twins. Both counts kept.
7. **Agent-office district rule**: moves owners NY/NJ from 97.9% to 97.3%. Keep the non-agent rule as the headline, or report both?
8. **(Moot under pass 4: the own-medallion rule replaced the medallion-lender rule.)** **c2 on the Lexis-only route.** c2 (lien fails, family share 0.5+) is kept only for liens in the DOS secured-party file. 2,109 Lexis-route DOS owner rows (1,466 owner-addresses) fail the same way and are dropped, many to core medallion lenders (Melrose CU, Progressive CU, Bay Ridge FCU, LOMTO FCU, MFIN). Options: add a Lexis-route c2 tier (kept), or keep as is (dropped). Not changed.

9. **One address, many spellings.** Kept DOS owner-addresses: 16,851. Same owner on Lexis at the same addr_key: 7,294. At the same `loose_key` only (54-11 vs 5411, 34TH vs 34): 7,190. A different address: 2,367. So many DOS-only points duplicate a Lexis point (HOSSAIN: 9 points, all 35-16 34th St Apt 1G, Astoria). Fold owner points by `loose_key`? Not done (changes every address count).

## Details

- **Whose address.** Owner addresses are the UCC debtor address at the filing date; median filing year 2009 (2008 before pass 4). Often a managing agent or garage, not a home; not current.
- **Owner linking.** Lexis: collateral names the owner's own medallion AND `name_match.match` is exact, set or subset. DOS v1 and v2: name match plus `ucc_lien_matched`, plus (pass 4) the lien, on Lexis by filing number, names a medallion on the owner's own list.
- **Lexis-backed basis.** Removed from the page in pass 4 (see below). `backed` stays per point for the DOS-only address toggle.
- **LLC rule.** LLC, L.L.C., L L C or LIMITED LIABILITY anywhere in the name (CHIPS AHOY TAXI LLC R, MULLAR TRANSPORT LLC TAXI SERVICE, NB LLC CAB).
- **Geocoding.** US Census batch geocoder on 24,963 addresses: exact 18,347 (73.5%), approximate 4,618 (18.5%), ZIP-level 1,870 (7.5%), unmapped 128. Wrong-state guard drops 8. Pass 2 sent 0 new addresses.
- **Districts.** County to district from 28 U.S.C. 81 to 131 (`data/0_raw/usc_title28_ch5_districts/`); problems in `district_problems.txt`.
- **Basemap.** OpenStreetMap tiles need a network connection; offline, points and district lines still draw.

## How to rerun

```
cd source/2_analysis/ucc_crash_comparison
~/Desktop/taxi-medallion-ucc-lexis/.venv/bin/python 261_run_owner_locations.py --skip-agents
```

- Drop `--skip-agents` to re-read the TLC files (about 5 GB on /Volumes/HDD).
- 261d sends only addresses not already in the cache.
- Screenshots: playwright script (pass-2 copy in the session scratchpad, `shot.py`); the page exposes `window.FIG116_VIEW` for it.

## Re-run 2026-10-12, pass 3: DOS medallion-lender rule

- **Rule** (William, 2026-10-12): a DOS v1 / v2 owner address counts only when the lien's secured party is a medallion lender.
- **Test used:** the existing lender medallion test of 261b (pass-2 lender audit), now also applied to owner rows (`dos_owner_medallion_test`, columns `med_test`, `qual_lender` in `owner_links_dos.csv`). No new rule.
  - Per lien number, v1 and v2 alike (v2 has no secured-party columns; it shares lien numbers with v1).
  - DOS secured-party file has a lender-class party (Organization or Individual): kept when lien, family or c2.
  - Not in the secured-party file, Lexis lien passes and names a lender family: kept (`lexis_lien`; lender from Lexis).
  - **c2 applies only to liens in the DOS secured-party file.** The Lexis-only route has no c2 tier: a Lexis lien that fails is dropped even when its Lexis lender family's share is 0.5 or more (open question 8).
  - Secured party unknown or not a lender: dropped, not counted.
- **Lexis owner records unchanged** (collateral names the medallion).
- **Lexis-backed switch kept:** still differs (all sources 20,711 owner addresses, Lexis-backed 13,458), because the rule does not stop same-name medallion owners.
- **Popups:** DOS owner points name the qualifying medallion lender, its source (NYS DOS UCC v1 or Lexis) and the test result.
- **Lenders, series LLCs, agents:** unchanged (1,337, 732, 95 points).
- **Screens:** `_screens/116_dosrule_*.png` (re-shot after the c2-scope note) (DOS owner popup, counts both bases, group rules, 900 px and 390 px full page). No console errors; no sideways page scroll at 1440, 900 or 390.

### DOS owner rows kept and dropped

| | DOS v1 | DOS v2 | All |
|---|---|---|---|
| lien (Lexis lien names a medallion) | 39,791 | 26,033 | 65,824 |
| family (lien not on Lexis, family share 0.5+) | 6,653 | 4,741 | 11,394 |
| c2 (Lexis lien non-medallion, family share 0.5+; open question 1) | 4,315 | 3,090 | 7,405 |
| lexis_lien (not in secured-party file, Lexis lien passes) | 19 | 16,293 | 16,312 |
| **Kept** | **50,778** | **50,157** | **100,935** |
| Dropped: not in secured-party file, Lexis lien fails or names no lender (no c2 tier on this route) | 1 | 5,098 | 5,099 |
| Dropped: fail_lexis | 987 | 638 | 1,625 |
| Dropped: no secured party in DOS or Lexis | 0 | 1,058 | 1,058 |
| Dropped: fail_family | 373 | 236 | 609 |
| Dropped: secured party only a filing agent, tax or other non-lender | 264 | 184 | 448 |
| **Dropped** | **1,625** | **7,214** | **8,839** |

- Owner-addresses: 21,596 kept, 4,263 dropped (16.5%) of 25,859.
- Of the 5,099 Lexis-route drops, 2,109 rows (1,666 liens, 1,466 owner-addresses) have a Lexis lender family share of 0.5 or more; on the DOS-file route these would be c2 and kept (`lexis_fail_c2_like` in `lender_stats.json`).
  - Top families (rows): Melrose CU 618, Progressive CU 239, Bay Ridge FCU 227, LOMTO FCU 221, MFIN 184, DePalma 127, Capital One 123, Rapid Funding 100.
- Row count: 109,774 rows in `owner_links_dos.csv`; 109,756 distinct on (entity, source, address, date, lien), the 18 extra differing only in match type or raw name. Points count distinct (entity, address), so the map is unaffected.
- Owners with any DOS row: 8,174 before, 8,122 after. Names mapped unchanged (10,332): every dropped owner keeps a Lexis or other address.

### Before and after (all sources)

| Group | Points, pass 2 | Points, pass 3 | Addresses, pass 2 | Addresses, pass 3 | Names mapped |
|---|---|---|---|---|---|
| Individual owners | 23,286 | 19,689 | 17,477 | 14,827 | 4,597 |
| Companies (LLC) | 2,617 | 2,547 | 1,577 | 1,518 | 1,615 |
| Companies (other) | 15,579 | 15,210 | 5,963 | 5,794 | 4,120 |
| **All owners** | 41,482 | 37,446 | 23,498 | 20,711 | 10,332 |
| All points | 43,646 | 39,610 | | | |

| Owners, latest non-agent address | SDNY | EDNY | NDNY | WDNY | D.N.J. | Other | NY/NJ share | Any address outside NY/NJ |
|---|---|---|---|---|---|---|---|---|
| Pass 2, all sources | 1,811 | 7,658 | 23 | 23 | 540 | 277 | 97.3% | 508 |
| Pass 3, all sources | 1,793 | 7,693 | 9 | 8 | 557 | 272 | 97.4% | 417 |
| Pass 2, Lexis-backed | 1,787 | 7,698 | 9 | 6 | 556 | 275 | 97.3% | 391 |
| Pass 3, Lexis-backed | 1,786 | 7,696 | 9 | 6 | 558 | 276 | 97.3% | 391 |

### HOSSAIN,MOHAMMAD (I01907, list medallion 5B67)

- DOS rows 181: lien 59, lexis_lien 38, family 4, c2 3 kept; lexis_fail 75, fail_lexis 2 dropped.
- Points 110 to 42 (41 DOS addresses + 1 Lexis-only); districts EDNY 85 to 33, SDNY 21 to 8, WDNY 3 to 1, NDNY 1 to 0.
- Still not clean: most kept liens fund other men's medallions with the same name (audit: 32 of the kept addresses; 8 name his 5B67, all 35-16 34th St, Astoria).
- Latest non-agent address moved from 89-11 153rd St 11432 (EDNY) to 491 Minnesota Ave 14215 (WDNY), from a DOS v2 row that passes the rule (not on Lexis for him). A stricter rule (lien names the owner's own medallion) would keep 16,851 of 21,596 owner-addresses; not applied.

## Re-run 2026-10-12, pass 4: own-medallion rule

- **Rule** (William, 2026-10-12): a NYS DOS v1 / v2 owner-address row counts only if its lien, looked up on Lexis by filing number, names at least one medallion on THIS owner's own list (list column `medallions`).
- Replaces the pass-3 medallion-lender rule for owner rows.
- **Collateral parse:** Lexis `ucc_records.csv` column `medallions` (= `parse_ucc_record.medallions_in_collateral`, the parse 261a uses), unioned over every record of the filing number.
- **Code:** 261b `dos_owner_medallion_test` (columns `med_test`, `own_lien_meds`, `qual_lender` in `owner_links_dos.csv`); 261f keeps `own_medallion` only.
  - A kept DOS row's medallions are now the owner's medallions its lien names (lien collateral, not the searched owner's list).
  - `qual_lender` = the lien's Lexis lender families (popup only, not a filter).
- **Unchanged:** Lexis owner records; lenders 1,337 points; series LLCs 732; agents 95 (`lender_stats.json` identical apart from the DOS owner keys).
- **Lexis-backed switch removed.**
  - 72,392 of 72,762 kept DOS rows (99.5%) sit on a lien 261a already links to the same owner on Lexis: identity is the same on both bases.
  - Pass-3 Lexis-backed vs pass-4 all sources, owners: SDNY 1,786 vs 1,784, EDNY 7,696 vs 7,702, NDNY 9 vs 8, WDNY 6 vs 6, D.N.J. 558 vs 556, Other 276 vs 276; NY/NJ 97.3% both.
  - The bases still differ in addresses (19,064 vs 13,458), but by spelling, not identity (open question 9).
- **Screens:** `_screens/116_pass4_*.png`.

### DOS owner rows kept and dropped

| | DOS v1 | DOS v2 | All |
|---|---|---|---|
| own_medallion (kept: lien names the owner's medallion) | 36,915 | 35,847 | 72,762 |
| Dropped: lien on Lexis, names no medallion | 6,810 | 10,083 | 16,893 |
| Dropped: lien not on Lexis | 7,156 | 6,126 | 13,282 |
| Dropped: lien on Lexis, names only other medallions | 1,522 | 5,315 | 6,837 |
| **Dropped** | **15,488** | **21,524** | **37,012** |
| Pass 3 kept / dropped | 50,778 / 1,625 | 50,157 / 7,214 | 100,935 / 8,839 |

- **Corrected in pass 6 (below):** this table matched DOS filing numbers to Lexis exactly, so pre-2003 6-digit DOS numbers (Lexis adds the 2-digit year) fell into "lien not on Lexis". With the fix: kept 76,215, not on Lexis 6,629 (not 13,282).

- Owner-addresses: 16,851 kept, 9,008 dropped (34.8%) of 25,859 (pass 3: 21,596 kept).
  - Dropped owner-addresses by best reason: no medallion 4,466, not on Lexis 2,606, other medallions only 1,936.
- Owners with any kept DOS row: 7,909 (pass 3: 8,122; before any rule: 8,174). Names mapped unchanged (10,332).

### Before and after (all sources)

| | Pass 3 | Pass 4 |
|---|---|---|
| Owner points | 37,446 | 33,411 |
| Owner addresses | 20,711 | 19,064 |
| Names mapped (owners) | 10,332 | 10,332 |
| All points | 39,610 | 35,575 |
| SDNY | 1,793 | 1,784 |
| EDNY | 7,693 | 7,702 |
| NDNY | 9 | 8 |
| WDNY | 8 | 6 |
| D.N.J. | 557 | 556 |
| Other | 272 | 276 |
| NY/NJ share | 10,060 / 10,332 (97.4%) | 10,056 / 10,332 (97.3%) |
| Any address outside NY/NJ | 417 | 391 |

| Group | Points, pass 3 | Points, pass 4 | Addresses, pass 3 | Addresses, pass 4 |
|---|---|---|---|---|
| Individual owners | 19,689 | 16,876 | 14,827 | 13,559 |
| Companies (LLC) | 2,547 | 2,498 | 1,518 | 1,492 |
| Companies (other) | 15,210 | 14,037 | 5,794 | 5,262 |

### HOSSAIN,MOHAMMAD (I01907, list medallion 5B67)

- DOS rows 181: own_medallion 19 kept; no_medallion 81, other_medallion 77, not_on_lexis 4 dropped (pass 3 kept 104).
- Points 42 to 9; districts EDNY 33 to 9, SDNY 8 to 0, WDNY 1 to 0.
- All 9 points are one apartment, 35-16 34th St Apt 1G, Astoria 11106, spelled 9 ways (1 Lexis-only, 2 DOS + Lexis, 6 DOS-only; one with ZIP 11110, one as 3156).
- Kept liens' lenders (Lexis): Mega Funding, Melrose CU, Capital One, NY Commercial Bank.
- Latest non-agent address: 491 Minnesota Ave, Buffalo 14215 (WDNY) to 3516 34th St, Astoria 11106 (EDNY, Lexis).

## Pass 3 (2026-10-12): what changed

- Precision key drawn with a neutral circle (the diamond is the company_other shape).
- Agent star: radius 7 to 8.5, white outline halo (ZIP-level stars keep the dark outline); spider legs light grey.
- Cluster radius shrinks with zoom; bubble hover lists per-group counts with shapes.
- Series LLC 732 split on the page as 730 series + 2 DePalma (stage 261b `spv_series`, `spv_no_medallion`).
- Counts unchanged (all rows identical to pass 2).

## Pass 5 (2026-10-12): redesign

William: "I don't understand what I'm looking at". Rebuilt through the template and builders only.

- **Dark page.** Tokens from the website-writer dark palette (page #0d0d0d, surface #1a1a19). Dark only by choice; no light toggle.
- **Basemap.** CARTO dark_all now returns an "API KEY REQUIRED" tile (checked 2026-10-12), so the page keeps OpenStreetMap tiles, darkened by a CSS invert filter. Tiles load in the headless renders.
- **Plain answer on top**, computed from the counts table (owners, latest non-agent address):
  - "97% of the 10,332 owners we could map last filed from New York or New Jersey: 75% Eastern District of NY (Brooklyn, Queens, Staten Island, Long Island), 17% Southern District of NY (Manhattan, Bronx, Westchester, lower Hudson Valley), 5% District of NJ, under 1% upstate NY (Northern and Western Districts), 3% elsewhere, mostly Florida."
  - "Mostly Florida": Florida districts hold 156 of the 276 owners outside NY and NJ (S.D. Fla. 101, M.D. Fla. 54, N.D. Fla. 1).
- **Court bar chart**, names at their latest address, scoped by the group chips. Owners: EDNY 7,702 (74.5%), SDNY 1,784 (17.3%), D.N.J. 556 (5.4%), S.D. Fla. 101 (1.0%), M.D. Fla. 54 (0.5%), D. Conn. 18 (0.2%), NDNY 8 (0.1%), WDNY 6 (0.1%), all other courts 103 (1.0%, 41 courts, expandable list). These equal the counts table (asserted in 261f).
- **Courts named in full everywhere**, with a plain area for the NY, NJ and CT courts and a county list on hover or focus. Areas live in 261e `AREA_CHECK`; each place named is asserted to be a county of that district.
- **Counts table** cut to group, names on our lists, names we could map, five NY/NJ courts, other courts. "Any address outside NY/NJ" moved to Details. Drivers in their own short box.
- **Scale views by zoom**, driven by the group chips: zoom 9 and out, counties shaded by names at their latest address; 10 to 13, hexagons about 0.5 mile across (exact at NYC's latitude); 14 and in, every address as group shapes (cluster radius 28 / 16 / 8 px). Shading: blue ramp, lighter = more names; classes are rounded quantiles, shown in the map legend.
- **Data added:** 261e writes `district_info.json` (full name, state, counties, area) and `county_shapes.json` (322 counties with a mapped address, 500k Census file, simplified 0.001 degrees), plus `map_county` in `addresses_located.csv` (60 Connecticut addresses carry pre-2022 county FIPS, placed in their 2023 planning region). 261f adds each point's `map_county` and each name's latest-point index; every other key of `owner_locations_map_data.json` is unchanged.
- **Details and method** folded at the bottom: sources, which address counts, DOS rows and Lexis, groups, any address outside NY or NJ, geocoding and courts, map views, open questions.
- **Visit log:** GoatCounter (`willcc.goatcounter.com`). `#me` stores a flag in this browser and its visits count as `/William/...`; `#me-off` clears it; a toast confirms. Tested headless on a fake host with every count request captured and aborted: plain load `/taxi-medallion-owner-map/index.html`, after `#me` `/William/taxi-medallion-owner-map/index.html` (also on a later plain load and after a typed hash change), after `#me-off` back to plain. No hit reached GoatCounter.
- **Screens:** `_screens/pass5/116_pass5_{1440,390}_*.png` (answer and bars, counties, NYC hexagons, street, street with all groups, details, counts and drivers, full page; 1440 also a search popup). No console errors; no sideways page scroll at either width (the counts table scrolls inside its own box at 390).

## Pass 6 (2026-10-12): verification fixes

- **Lien-number format fix (261b `lexis_lien_key`).** Pre-2003 NYS DOS lien numbers have 6 digits; Lexis stores the same filing as yy(filing year) + the 6 digits (DOS 030259 filed 2002-02-07 = Lexis 02030259). The lookup now tries the exact number, then yy + number, then the number without a trailing '-N' check digit.
  - Owner rows matched by yy + number: 6,653 (`dos_owner_lien_keys_normalised`). Check-digit strip: 0 of 105 rows (Lexis holds no such filing; extract ends 2026-07-31).
  - Same key now used by the lender medallion test: 4,292 lender rows re-keyed (`lender_lien_keys_normalised`).
  - Matches an independent rebuild (same-year only) exactly.

| DOS owner rows | Pass 4 | Pass 6 |
|---|---|---|
| own_medallion (kept) | 72,762 | 76,215 |
| no_medallion | 16,893 | 19,427 |
| other_medallion | 6,837 | 7,503 |
| not_on_lexis | 13,282 | 6,629 |
| Owner-addresses kept / dropped | 16,851 / 9,008 | 17,485 / 8,374 |
| Owners with a kept DOS row | 7,909 | 7,999 |

| | Pass 4 | Pass 6 |
|---|---|---|
| Owner points | 33,411 | 33,803 |
| Owner addresses | 19,064 | 19,318 |
| DOS-only spelling points | 9,546 | 9,938 |
| Points at an agent office | 6,432 | 6,482 |
| SDNY | 1,784 | 1,786 |
| EDNY | 7,702 | 7,700 |
| NDNY, WDNY, D.N.J., Other | 8, 6, 556, 276 | unchanged |
| NY/NJ share | 97.3% | 97.3% |
| Lender points / names | 1,337 / 578 | 1,339 / 579 |
| Lender links by test (lien, family, c2) | 58,558, 8,643, 5,970 | 62,070, 4,620, 6,550 |
| Lender points by test (lien, family, c2) | 1,154, 136, 47 | 1,221, 58, 60 |
| HOSSAIN,MOHAMMAD (I01907) kept DOS rows / points | 19 / 9 | 21 / 9 (same apartment) |

- Two owners change court: GRIP SERVICE CORP and TOBA CAB CORP, EDNY to SDNY.
- c2 lender points (open question 1) rise 47 to 60.
- **Court labels.** D.D.C. read "District of District of Columbia"; fixed in `_districts_28usc.py` (also used by 262c, not re-run). Florida districts get a plain area in 261e `AREA_CHECK` (each city asserted to be a county of that district): S.D. Fla. Miami, Fort Lauderdale, West Palm Beach; M.D. Fla. Orlando, Tampa, Jacksonville; N.D. Fla. Tallahassee, Pensacola, Gainesville.
- **Page.**
  - TLC, NYS DOS, NYS DMV and Lexis spelled out once (header, answer bullets, Details > Sources, Drivers box).
  - Drivers box in plain words ("with a name on file", "taxi drivers in crashes (counted once per crash)", no "case key").
  - Series LLC row shows 735 names on the owner lists with a footnote: mapped 732 = 730 NY DOS corporations + 2 DePalma vehicles.
  - Street-level toggles count "points" and say a point is one owner at one address; DOS-only toggle relabelled "Address spellings found only on NY DOS liens (often a Lexis address spelled differently)".
  - Bar sub-heading "Latest address: all owners"; one build stamp (footer); source path dropped from the footer.
  - Counts table: group column sticky, caption above the scroller, "scroll sideways" hint below 40em, tap a court for its counties (phone).
  - Popups pan clear of the zoom buttons (`autoPanPaddingTopLeft` [56, 16]; narrower on phones).
  - Shading ramp lifted: #306bc0, #4385dc, #68a2ea, #9cc5f3, #d3e6fb (lowest step 3.2:1 on the map background, was 2.08:1), fill opacity 0.9.
- **Not changed (William's call).** Headline wording "last filed from" (the address is the latest non-agent UCC debtor address, and the debtor did not file it); folding DOS-only spellings by `loose_key` (open question 9).
- **Screens:** `_screens/pass6/116_pass6_{1440,390}_*.png`; 390 adds the search popup and a tapped court. JS parses; no console errors apart from GoatCounter skipping a local file; no sideways page scroll at 1440 or 390.

## Status

Built 2026-10-12, passes 2 and 3 the same day; re-run the same day with the DOS medallion-lender rule (pass 3), the own-medallion rule (pass 4), the redesign (pass 5) and the verification fixes (pass 6). Published on GitHub Pages at William's request. Passes 4 to 6 not committed or pushed. Nothing sent to Ed.
- A stale pass-1 copy (names with addresses) sits outside the project at `~/Sites/taxi-map/` (synced by `~/Sites/taxi-map-bin/sync_taxi_map.sh`), served by `python -m http.server 8734` on 127.0.0.1 and proxied tailnet-only by Tailscale at `/taxi-map`. Not created by stage 261; left as is.
