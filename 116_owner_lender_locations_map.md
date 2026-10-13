# Figure 116: medallion owners by bankruptcy court, the addresses on their UCC lien filings (interactive map)

## Goal

- Ed, 2026-10-12: "Do all taxi drivers/companies live in New York and New Jersey? I need to know their locations in order to request bankruptcy data from the appropriate court."
- Map every owner, lender and TLC managing agent we hold an address for, and give each one's federal (bankruptcy) district.

## Output

- `116_owner_lender_locations_map.html`: published on GitHub Pages (https://williamclintc.github.io/taxi-medallion-owner-map/), names and addresses shown. Leaflet 1.9.4 and markercluster 1.5.3 are inlined. Data is in the sibling `116_owner_lender_locations_map_data.js` (9.90 MB since the pass 8 fixes: court polygons added; 9.58 MB in pass 8 with the city and ZIP tables, state polygons and ZIP centroids; 9.22 MB in pass 7; county shapes and court names added in pass 5).
- Pass 7: a second data file, `116_owner_lender_locations_map_filings.js` (11.55 MB since pass 8, 11.38 in pass 7; stage 261h from stage 262's outputs), holds the medallions-with-liens view. Since pass 8 the page opens on that view and loads the file on page load (pass 7 loaded it on the first switch). The view's code is `_owner_loc_page_filings.js`, inlined by 261g.
- Six groups, colour by family (owners blue, lenders orange, agents teal) and shape by group, used the same way on the map, in the legend, in popups, chips and the counts table (pass 5; pass 1 to 4 used six Okabe-Ito hues). Points and names mapped are pass-7 values (one point per place; people, not list rows); earlier values in each pass section:

| Group | Shape | Colour | Points | Names mapped |
|---|---|---|---|---|
| Individual owners | circle | #7fb3ef blue (mid) | 12,896 | 4,730 |
| Companies (LLC) | square | #c4dcf8 blue (light) | 2,028 | 1,612 |
| Companies (other: INC, CORP, etc.) | diamond | #2f7fe0 blue (dark) | 10,967 | 4,119 |
| Lenders | triangle | #d95926 orange | 1,295 | 579 |
| Lender-held series LLCs (730 series + 2 DePalma acquisition vehicles) | down triangle | #f4a27c orange (light) | 732 | 732 |
| TLC managing agents | star (light halo, larger) | #199e70 teal | 95 | 82 |

- Drivers are not on the page since pass 8 (the Drivers box was removed at William's request; its numbers are in the pass 8 section).

- Colours (pass 5, dark page): base hues #3987e5, #d95926, #199e70 pass the dataviz validator (dark, surface #1c1d20, all pairs: CVD worst 9.4 deutan, normal 20.9, all at least 3:1). Owner shades #2f7fe0 / #7fb3ef / #c4dcf8 and lender shades #d95926 / #f4a27c pass the ordinal check (adjacent lightness gap at least 0.06, at least 4.2:1 on the surface). Shape carries the group within a family.
- Markers are canvas paths (no DOM per point). Pass 3: 39,610 points loaded in about 5 s headless (pass 4: 35,575 points); a layer toggle takes 0.1 to 0.3 s.
- Cluster bubbles: one cluster layer for all groups; each bubble is a donut whose ring shows the group mix.
- Hover a bubble: per-group counts with each group's shape (readable without colour).
- Cluster radius shrinks with zoom (44 px, 20 px from zoom 15, 6 px from zoom 17); same-address stacks still cluster and spiderfy.
- Toggles (pass 8): the view switch opens on Medallions with liens (every filing), Owners by name second; Group by (Auto by zoom, State, Court, County, City, ZIP, Hexagon 0.5 mi, Address) over the map in both views, with a ranked By-unit table under the map and hover highlight on every shaded level; the owners Measure opens on Every address on file and the filings Count on Medallions, every address; shading on a blue-to-red ramp. Toggles (pass 7): each group (pass 5: owners on by default, scoping the court bars and the map); the measure (Most recent address / Every address on file); source chips for owner points (Lexis only 9,819, NY DOS only 2,508, both 13,564; they replace the pass-6 toggle for spellings found only on NYS DOS liens); owner points at a managing-agent office (3,499); district lines. Pass 4 removed the All sources / Lexis-backed only switch (one basis).
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

## Counts (stage 261f, re-run 2026-10-12 pass 6, own-medallion rule with the lien-number fix; superseded by pass 7 below)

Pass-6 values, kept as the record. The page now shows the pass-7 counts (section "Re-run 2026-10-12, pass 7").

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
- **Unmatched addresses (pass 9).** `_owner_loc_fallback.place_unmapped` puts an address the geocoder could not place at the middle of our Census-matched addresses with the same ZIP (3 or more of them; 00000 is no ZIP), else the same city, else at the Census 2023 place point; precision ZIP-level or city-level ("city area only" on the page). Owners 143 of 153 placed, filings 99 of 106. City units of unmatched addresses come from `city_unit` (the filed city checked against the ZIP's Census cities).
- **Districts.** County to district from 28 U.S.C. 81 to 131 (`data/0_raw/usc_title28_ch5_districts/`); problems in `district_problems.txt`.
- **Basemap.** OpenStreetMap tiles need a network connection; offline, points and district lines still draw.

## How to rerun

```
cd source/2_analysis/ucc_crash_comparison
~/Desktop/taxi-medallion-ucc-lexis/.venv/bin/python 261_run_owner_locations.py --skip-agents
```

- Drop `--skip-agents` to re-read the TLC files (about 5 GB on /Volumes/HDD).
- 261d sends only addresses not already in the cache.
- 261e and 261h read the Census place gazetteer (`data/0_raw/census_geo_2023/place_gaz/`) and 262's `filing_addresses.csv` through `_owner_loc_fallback` (pass 9); 262 must have run first.
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

## Re-run 2026-10-12, pass 7: two measures, filings layer, people, places, guards

William's design (2026-10-12). Everything through stages 261 and 262 and the template; new modules `_lexis_blocks.py` (262a's block reader, moved verbatim, 262a output unchanged on a 5,045-record sample), `_owner_loc_guards.py`, `_owner_loc_people.py`, `_owner_loc_measures.py`, `_owner_loc_panels.py`, `261h_build_filing_layer.py`, `_owner_loc_page_filings.js`.

### 1. Two measures

- **Most recent address**: one court per name, the address on its newest lien filing. Bars (default), headline sentence 1, counts-table block 1.
- **Every address on file**: a name counts in every court it ever had a lien address in; the full set of courts worth requesting records from (a bankruptcy from 2012 on could have been filed where the owner lived then). Bars (switch), headline sentence 2, counts-table block 2 with "only ever NY or NJ" and "any address elsewhere".
- Headline as rendered (1440 and 390 the same):
  - "Latest address: 98% of the 10,461 owners we could map had their latest address on a UCC lien filing in New York or New Jersey: 77% Eastern District of NY (Brooklyn, Queens, Staten Island, Long Island), 17% Southern District of NY (Manhattan, Bronx, Westchester, lower Hudson Valley), 5% District of NJ, under 1% upstate NY (Northern and Western Districts), 2% elsewhere, mostly Florida."
  - "Every address on file: 96% have only ever had NY or NJ addresses on their lien filings; 4% (392 owners) have had at least one address elsewhere, most often Southern District of Florida (124), Middle District of Florida (66), District of Connecticut (27)."
- Wording: "filed from" is gone from the title, headline and bar sub-headings (the lender files the lien; the owner's address is the debtor address on it).

| All owners | N | SDNY | EDNY | NDNY | WDNY | D.N.J. | Other | NY/NJ |
|---|---|---|---|---|---|---|---|---|
| Pass 6, latest non-agent address (names) | 10,332 | 1,786 | 7,700 | 8 | 6 | 556 | 276 | 10,056 (97.3%) |
| Pass 7, most recent address (people and firms) | 10,461 | 1,745 | 8,012 | 8 | 4 | 496 | 196 | 10,265 (98.1%) |
| Pass 7, every address on file (name-court pairs) | 10,461 (12,774 pairs) | 2,631 | 8,946 | 24 | 11 | 759 | 403 | only ever NY/NJ 10,069 (96.3%); elsewhere 392 (3.7%) |

- Every address on file, other courts (All owners): S.D. Fla. 124, M.D. Fla. 66, D. Conn. 27, E.D. Pa. 21, M.D. Pa. 15, N.D. Ga. 13, C.D. Cal. 10, W.D. Wash. 9. Most recent address: S.D. Fla. 64, M.D. Fla. 37, D. Conn. 11, M.D. Pa. 10, E.D. Pa. 7.

| Group, pass 7 | Rows | People (est.) | Mapped | Latest: SDNY | EDNY | NDNY | WDNY | D.N.J. | Other | Ever: SDNY | EDNY | NDNY | WDNY | D.N.J. | Other | Only NY/NJ | Elsewhere |
|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|
| Individual owners | 5,117 | 5,250 | 4,730 | 481 | 3,748 | 7 | 2 | 332 | 160 | 817 | 4,199 | 19 | 6 | 468 | 326 | 4,415 | 315 |
| Companies (LLC) | 3,952 | 3,952 | 1,612 | 329 | 1,207 | 1 | 0 | 68 | 7 | 349 | 1,241 | 2 | 0 | 78 | 10 | 1,602 | 10 |
| Companies (other) | 5,158 | 5,158 | 4,119 | 935 | 3,057 | 0 | 2 | 96 | 29 | 1,465 | 3,506 | 3 | 5 | 213 | 67 | 4,052 | 67 |
| **All owners** | 14,227 | 14,360 | 10,461 | 1,745 | 8,012 | 8 | 4 | 496 | 196 | 2,631 | 8,946 | 24 | 11 | 759 | 403 | 10,069 | 392 |
| Lenders | n/a | n/a | 579 | 162 | 242 | 12 | 3 | 50 | 110 | 176 | 253 | 13 | 3 | 58 | 131 | 457 | 122 |
| Lender-held series LLCs | 735 | n/a | 732 | 478 | 254 | 0 | 0 | 0 | 0 | 478 | 254 | 0 | 0 | 0 | 0 | 732 | 0 |
| TLC managing agents | n/a | n/a | 82 | 17 | 65 | 0 | 0 | 0 | 0 | 18 | 66 | 0 | 0 | 0 | 0 | 82 | 0 |

### 3. Latest-address rule (272-owner audit)

- Latest = the point with the newest filing date of any kind (continuation and termination events count), managing-agent offices included, no fall-back to an older address elsewhere; same date: a NY or NJ address first, then more records, then the key (`_owner_loc_measures.latest`, rule `newest`).
- 262's guards on the owner points (`_owner_loc_guards`, 262's own code through `_lexis_blocks`): 135 link rows dropped (lender address copied into the debtor block 121, C/O a lender 9, debtor is a lender 5; Lexis 81, DOS v1 36, DOS v2 18), 53 owner-address spellings, 16 of them with no other row (15 places), 44 owners.
- 261a reads debtor blocks with 262's strict reader: an address ends at a blank line or the next label, one row per address in a block, ZIP-only addresses kept. Lexis links 87,126 to 90,336; 187 old keys (merged or glued spellings) gone, 125 new keys, 61 of them ZIP-only (165 link rows carry a ZIP-only key; 4 in pass 6). Geocoder: 55 answers copied from 262's cache, 1 address sent (5131 Northern Blvd, Woodside 11377, exact match), a spelling of 51-31 Northern Blvd already in the cache; 261d now looks both caches up by place_key before sending (verification fix below).
- **Owners changing court, pass-6 rule to pass-7 rule on the same points: 322 of 10,461** (SDNY to EDNY 125, EDNY to SDNY 91, S.D. Fla. to EDNY 29, D.N.J. to EDNY 27, M.D. Fla. to EDNY 11, D. Conn. to EDNY 6, D.N.J. to SDNY 5, M.D. Fla. to SDNY 4, E.D. Pa. to EDNY 4, N.D. Ga. to EDNY 3, and 15 more). Pass-6 rule on the pass-7 points: SDNY 1,767, EDNY 7,884, NDNY 9, WDNY 6, D.N.J. 528, Other 267.
- Owners whose latest point is an agent office: 2,285 (909 have no other place).

### 4. Shared-name rows split into people

- Individual rows with 2+ medallions: 135 tested. Clusters = medallions whose own Lexis filings, inside each medallion's holding dates (stage 159 spells), share a loose-key address; hub addresses used by 5+ owner rows (389 keys) excluded; a row splits when two clusters were held at the same time.
- **80 rows split into 213 people** (largest 11); 45 rows not split, flagged "may be more than one person"; 2 rows with two sequential clusters kept as one (moved house). Individuals: 5,117 rows, about 5,250 people; all owners 14,227 rows, about 14,360 people and firms. Companies never split.
- A medallion with no filing inside its holding dates is shown with the row's first person, marked "placed by default" (353 link rows); such points drive neither measure when the person has a real point.
- Verified: SINGH,BALWINDER (I04260) splits into 9 (3C57, 4D98, 6A95, 6B87, 6C79, 7C86, 8E34, 8E86, 9Y14; 4C31 placed by default); HOSSAIN,MOHAMMAD (I01907) stays one entity.
- Counts table reads "People (estimated)" beside the list rows; popups and search results say "one of N people with this name".

### 5. Duplicate spellings folded into one point

- One point per (entity, place), place = `_owner_loc_common.place_key`: loose_key (C/O, house-number hyphen, ordinals, unit designators) plus a suffix glued to the number split (34ST = 34 ST) and a trailing unit (1G) dropped; ZIP kept. The point takes the best-geocoded spelling's coordinates; the popup lists every spelling with its source.
- Points: pass 6 35,969 (owners 33,803, lenders 1,339, series LLCs 732, agents 95) to pass 7 28,013 (owners 25,891 from 33,710 spellings at 14,858 places; lenders 1,295 from 1,339; series LLCs 732; agents 95 from 130).
- HOSSAIN,MOHAMMAD (I01907): 9 points to 3 (35-16 34th St 11106 with 7 spellings; the typo 3156 34th St; one spelling with ZIP 11110).
- The "spellings found only on NY DOS" toggle is replaced by source chips (owner points): Lexis only 9,819, NY DOS only 2,508, both 13,564.

### 6. Loan-filings view (stage 262 data, 261h)

- Switch under the title: "Owners (people and firms)" / "Loan filings (every medallion loan, all years)". One-liner: no name matching, the medallion number in the loan's collateral is the key.
- Rows 137,038 located (guard-free, districted), 20,395 addresses, 113,358 filings, 13,776 medallions (13,154 on the TLC list; 622 off-list medallions and their 506 own rows off by default, toggle). County shading, hexagons and street points of debtor addresses (circle person, diamond company, violet); popup per address lists every row under the filters (debtor as filed, date, lender, medallions, filing type, record id).
- Court bars on the three bases, recomputed in the browser for the year filter (default all years; one click "2012 and later") and the TLC toggle. All-years numbers equal 262d's `summary.json` (asserted in 261h, checked headless for all six tables).
- Medallion search draws one medallion's address history as numbered stops with dates (5B67: 27 rows, 7 addresses, 1 court).

| TLC-listed medallions | Basis | N | SDNY | EDNY | NDNY | WDNY | D.N.J. | S.D. Fla. | M.D. Fla. | D. Conn. | Other | NY/NJ |
|---|---|---|---|---|---|---|---|---|---|---|---|---|
| All years | A rows | 136,532 | 22,938 | 103,937 | 114 | 49 | 7,219 | 632 | 353 | 153 | 1,137 | 98.3% |
| All years | B medallions, every court | 13,154 (20,027 pairs) | 5,538 | 12,103 | 64 | 43 | 1,351 | 229 | 110 | 52 | 537 | only ever NY/NJ 12,279 (93.3%); 875 elsewhere |
| All years | C medallions, latest filing | 13,154 | 2,487 | 9,835 | 10 | 7 | 548 | 87 | 50 | 20 | 110 | 98.0% |
| 2012 and later | A rows | 29,117 | 3,946 | 21,874 | 25 | 13 | 2,360 | 262 | 154 | n/a | 418 | 96.9% |
| 2012 and later | B medallions, every court | 10,906 (12,532 pairs) | 2,599 | 8,812 | 13 | 9 | 672 | 141 | 75 | 30 | 181 | only ever NY/NJ 10,489 (96.2%); 417 elsewhere |
| 2012 and later | C medallions, latest filing | 10,906 | 2,083 | 8,070 | 7 | 6 | 493 | 78 | 48 | 19 | 102 | 97.8% |

- 2012 and later, A rows: D. Conn. is not among the three largest other courts (M.D. Pa. 65 is), so the "Other" cell holds it.
- Every medallion named (off-list included), all years: the "A rows / B ever / C latest" columns of stage 262's `summary.md` (A 137,038 rows, 98.3% NY/NJ; B 20,670 pairs, 95.1%; C 13,776 medallions, 97.5%); the page reproduces them with the toggle on.
- Filings headline as rendered, all years: "All years, TLC-listed medallions: 98% of 136,532 filing rows carry a debtor address in New York or New Jersey, and the most recent filing places 98% of 13,154 medallions there. Counting every court a medallion ever had an address in: 93% only ever NY or NJ, 7% (875) at least one elsewhere, most often Southern District of Florida (229), Middle District of Florida (110), District of Connecticut (52)."

### Checks

- JS parses (every script block); no console errors apart from GoatCounter skipping a local file; no sideways page scroll at 1440 or 390 in either view; GoatCounter tag and `#me` logic unchanged.
- 261f asserts the latest-point districts and the every-address districts reproduce the counts table for every group; 261h asserts the three bases against 262d.
- Screens: `_screens/pass7/116_pass7_{1440,390}_*.png` (answer and both measures, filings bars with the 2012 filter, filings counties, medallion path, split-name popup, HOSSAIN popup, street level both views, counts, details both views, full page).
- Not done: the people split uses Lexis in-spell addresses only (NYS DOS rows place medallions, never split them); off-list medallions are drawn but not named in the filings headline; the rows the audit found with a lender's city and ZIP in the debtor block (CABLE CAB CORP) stay as 262 left them.

### Verification fixes (pass 7, same day)

Sixteen verifier findings on pass 7; all fixed through the stages, the template and this note, then stage 261 re-run (`--skip-agents`; every count identical to pass 7, 261d sent nothing). None changes a number on the page.

- **Note.** Output table and the Toggles bullet refreshed to pass-7 values (asserted equal to `owner_locations_map_data.json`); the pass-6 Counts section stamped as superseded; pass-6 points 35,575 to 35,969 (here and in `FIGURES.md`); "125 new (165 ZIP-only keys)" to 125 new keys, 61 ZIP-only, 165 link rows; HOSSAIN's main point 6 to 7 spellings; "16 places" to 16 spellings at 15 places (261f now writes `places_only_guarded`; the page's guards line says both).
- **Geocoder (261d).** Both caches are looked up by `place_key` before anything is sent, so a re-spelled cached place (5131 vs 51-31 Northern Blvd) takes the cached answer under the new key (`copy_by_place`; the row keeps the spelling sent and the Census answer). Unit-tested: with the 5131 row removed it is filled from the 51-31 row, not sent. This re-run: copied by place 0, sent 0.
- **Counts table.** The right edge fades and a hint ("Scroll the table sideways for the rest of the columns") shows whenever columns are off-screen, at any width, and clears at the end of the scroll (1440: scroll width 2,124 px in a 1,278 px box; 390: 1,965 in 324). Caption names the two blocks left to right.
- **Wording.** Meta description no longer says "filed from". Owners facts: "Addresses found by matching owner names to the debtor names on the liens (word order ignored); the Loan filings view skips the name match and keys on the medallion number instead". Switch note: "Not by owner name: the medallion number in the loan's collateral is the key". Filings headline N: "filing rows naming a TLC-listed medallion" (or "naming a medallion" with the toggle on) and "TLC-listed medallions". Filings facts: "Medallions named: 13,794; with a located address 13,776, of which 13,154 on the TLC worklist and 622 off it". Basis button "Medallions, every court"; its note says "ever" only with the year filter off, else "in the years chosen" (latest-filing note likewise).
- **Medallion path.** Stops closer than 22 px at the current zoom share one marker labelled with their numbers ("5 to 7", "5, 7"), tooltip listing each stop's date and address; redrawn on zoom. 5B67 at zoom 11: 1, 2, 3, 4, "5 to 7"; at zoom 14: "5, 7" and 6 apart.
- **Phone.** The map legend hides while a popup is open under 640 px and returns on close; popups under 640 px take a max height of 60% of the map and scroll inside (SINGH,BALWINDER 3C57 popup bottom at 377 px in a 558 px map; the 6814 5th Ave filings popup 266 in 558). Tap targets: view switch, measure, count, year inputs and buttons, map views, Clear at 44 px; chips 40 px.
- **Screens:** `_screens/pass7/116_pass7v_{1440,390}_*.png` (answer and bars, counts table with the cue and scrolled to the end, split popup, street level, filings bars 2012 and later on the every-court basis, medallion path at two zooms, a filings popup at 390, details, full page). JS parses (four script blocks); no console errors apart from GoatCounter skipping a local file; no sideways page scroll at 1440 or 390 in either view; all six all-years bases equal `summary.json`; GoatCounter tag and `#me` logic unchanged.

## Pass 8 (2026-10-12): Group by levels, landing on the filings view, every-address default, blue-to-red ramp, hover highlight, drivers box removed

William's messages, 2026-10-12, verbatim: "in the map can you give an options of viewing it by state by city by county ,e tc. basically wia t tn to have ht eabilitity to gorup the blues into different subsceiton"; "i think wha tiw ant for hte visualization is to focus on the medallion with liens on them in the ucc and just gather their adressed and plot them"; "can you also put more sutbacne to the color maybe blue to red. i don think blue to white sufficiently captires the entirety of the datset"; on the Drivers box, "remove this i dont understand why we need this"; "when i pass throguh poritons of the map make it highlight as well the number"; on the Counts by group table, "what is the lenders name son our lsit columgn row entry why is it no list wha tis a tlc managin eagent i have so many question why are therese blank"; "for this one as well make sure to sort by owners we are arpticlaulyr intersted in owners so owners by descneding order is the order"; "make ht edefault every adres son file please"; "this shoudl be the default and hte tbale shsoudl be sroted with this descending All owners every address". Copy on the page is the builder's draft; a website-writer copy pass rewrites every reader-facing string after this build.

### What changed

- **Landing view.** The page opens on "Medallions with liens (every filing)" (stage 262's rows, no name matching); "Owners by name (people and firms)" is the second switch. Title "Medallions with UCC liens: where their loan filings place them"; subtitle and meta description lead with the medallions. The filings data file loads on page load (2.9 s headless to ready, 1440 and 390).
- **Defaults.** Owners Measure = Every address on file (a name counts once in every unit it has a place in); filings Count = Medallions, every address (basis B: a medallion counts once in every unit it ever had a debtor address in). Year filter all years; TLC-listed only. The latest-address measure and bases C and A stay one click away; each headline reports them in its second sentence.
- **Group by** (segmented control over the map, both views, 44 px targets, arrow keys): Auto (by zoom: counties under zoom 10, 0.5-mile hexagons 10 to 13, addresses 14 and in) | State | Court | County | City | ZIP | Hexagon (0.5 mi) | Address. A pinned level draws that unit at every zoom until Auto is picked again. Each view supplies `unitCounts(level)` ({key: {n, mix, label, latlng}}, N) and `hexItems()`; the shared map code draws polygons (State, Court, County), bubbles (City, ZIP), hexagons and the address cluster.
- **Counts follow the measure.** Owners: latest = each name once at its latest point; ever = a name once per unit it has a measure-ok point in (hexagons dedupe names per hexagon too). Filings: A = rows; B = medallion-unit pairs; C = medallions at their most recent filing (ties as 262d). The year and TLC filters apply to every level.
- **Ranked table "By <unit>"** under the map: unit, count, share of N, the group mix (owners: the groups ticked; filings: person and company rows for A, filing rows for B and C), largest first, top 20 then "Show all N"; N line names the measure; a row click zooms the map to the unit (fitBounds for polygons, two zoom levels in on a bubble); hovering a row lights the unit on the map. Hidden at the Hexagon and Address levels (one line says why). Phone: horizontal scroll box with the same fade and hint as the counts table.
- **Hover highlight** on every shaded level (the shade pane now sits above the default overlay canvas, with the district lines and the medallion path in a pane above it that lets the pointer through, and the shade pane lets the pointer through at the address level; before this the overlay canvas swallowed every hover on a shaded unit): white outline (weight 2.5), full opacity, brought to front; a tooltip whose first line is the count at 1.4 em, then the unit name and the measure, then the mix; the matching ranked-table row lights up (and scrolls into view inside the table); on the Court level the court bar lights up too. Pointer out restores everything. Touch: a tap highlights and shows the number, a second tap zooms. Address level keeps its popups.
- **Colour ramp.** The light-blue-to-white ramp is replaced by `RAMP = #2a5bd7, #3f86f5, #7b5cf0, #b43fd3, #e63a6e, #f0463a, #ff8a1f` (deep blue, blue, violet, magenta, rose red, red, hot orange), used for every shaded unit and the legend. Breaks: quantiles of the nonzero counts at 0.2, 0.4, 0.6, 0.8, 0.9, 0.97, rounded to 1-2-5 numbers, deduplicated (fewer classes when the data has fewer distinct values); k classes take k colours spread over the ramp, the first always the deep blue. The group colours (owners blue, lenders orange, agents teal; filings violet) are unchanged; the legend says what the ramp counts ("Medallions per city, every address"; "Names per state, every address on file") and the hint says blue = fewest, red = most.
- **Drivers box removed** (panel, markup, CSS, the `drivers` key of the data file, `drivers()` in `_owner_loc_panels`). Its last numbers, for the record: TLC taxi driver licences 56,648, 51,508 with a name on file, none with an address; crash reports 2018 to 2024, driver licence state NY 17,269 (94.0%), NJ 788 (4.3%), PA 108 (0.6%), other or blank 209 (1.1%) of 18,374 taxi drivers in crashes; no crash county (NYS Case Information has no case key).
- **Counts by group, split in two, no placeholder cells.** Table 1: the owner groups (Individual owners, Companies (other), Companies (LLC); largest first by names mapped) with the list columns (names on the TLC owner lists, people estimated, names mapped) and the court blocks; All owners at the bottom. Table 2: Lender-held series LLCs, Lenders, TLC managing agents (largest first by names mapped) with only the columns they have (names mapped and the court blocks). Every address on file is the first (left) block, Most recent address second. Court columns ordered by the All owners count under Every address on file, largest first (EDNY 8,946; SDNY 2,631; D.N.J. 759; NDNY 24; WDNY 11). No "no list", "n/a", dash or empty cell anywhere on the page (popups say "lender not named", "none named", "not tested"). A gloss line under each table and a title attribute on each chip name the groups (draft text; the copy pass rewrites it).
- **Sort by owners, descending.** Court bars, the court columns, the Other courts list (rows by the All owners every-address count, every-address columns first) and the ranked table all sort by count, largest first; the All other courts row last. The filings view sorts by medallions the same way.

### Group-by levels: how the units are built

- **State** key = the first two digits of the 2023 county FIPS the map shades (the county that sets the court); an address with no county takes its state code as filed (`state_shapes.json` maps code to FIPS and name). Polygons: `cb_2023_us_state_20m` (public Census cartographic boundary file, downloaded 2026-10-12 to `data/0_raw/census_geo_2023/state20m/`), Douglas-Peucker 0.01 degrees, rounded to 1e-3, every state plus DC and PR (52 features, 130 KB), stage 261i.
- **Court**: one shape per court (`court_shapes.json`, stage 261e: every `cb_2023_us_county_20m` county dissolved by district into closed rings, `_shapefile_lite.dissolve_polygons`, 91 districts, 172 rings, 16,315 points, 0.32 MB, drawn with the even-odd fill), shaded by the court's total; tooltip = the full court name, its area, the count; the district boundary lines stay on top. Pass 8 as first built shaded only the counties holding an address, so a court read as a patchwork (fixed the same day, see Pass 8 fixes).
- **County**: the 2023 county the map shades (as before), now on the measure chosen.
- **City** unit = (state, city label), `_owner_loc_geo_units.city_of`: the city in the Census geocoder's matched address ("street, CITY, ST, ZIP") when the address matched; else the filed city upper-cased, commas to spaces, trailing punctuation removed, a trailing state word equal to the address's state removed (BROOKLYN NY -> BROOKLYN). Both then go through the same normalisation (pass 8 fix; as first built the Census city was taken as is, so S OZONE PARK from the geocoder sat beside SOUTH OZONE PARK): the fold table below (for filed cities looked up before and after the punctuation strip), then USPS-style abbreviations expanded token by token (a leading S, N, E, W to SOUTH, NORTH, EAST, WEST; ST before another word to SAINT; HTS and HGTS to HEIGHTS; PK to PARK; VLG to VILLAGE; HL to HILL; CTY to CITY; BCH to BEACH; ISL to ISLAND; a trailing dot on the abbreviation ignored), then the fold table again. Drawn as a bubble at the mean position of the unit's distinct addresses; label "Brooklyn, NY" (title case in labels only).

| Filed spelling | Folds to |
|---|---|
| NEW YORK CITY, NYC, N.Y.C., N.Y.C, N.Y., N.Y, MANHATTAN, NEW YORK NY | NEW YORK |
| BKLYN, BKLN, BROOKLYN NY | BROOKLYN |
| L.I.C., L.I.C, LIC, L I C, LONG ISL CITY, LONG ISLAND CTY, LI CITY | LONG ISLAND CITY |
| S.I., S.I, STATEN ISL, STATEN IS | STATEN ISLAND |
| FLUSHING NY | FLUSHING |
| THE BRONX, BRONX NY | BRONX |
| QUEENS VLG | QUEENS VILLAGE |
| S RICHMOND HILL, S. RICHMOND HILL | SOUTH RICHMOND HILL |
| S OZONE PARK, S. OZONE PARK | SOUTH OZONE PARK |
| E ELMHURST, E. ELMHURST | EAST ELMHURST |
| JAMAICA ESTATE | JAMAICA ESTATES |
| JACKSON HTS | JACKSON HEIGHTS |
| FOREST HILL | FOREST HILLS |
| REGO PK | REGO PARK |
| KEW GARDEN | KEW GARDENS |
| LONGISLANDCITY | LONG ISLAND CITY |
| QUEENSVILLAGE | QUEENS VILLAGE |

Abbreviations expanded after the fold (examples from the data): S OZONE PARK, S OZONE PK, S. OZONE PARK to SOUTH OZONE PARK; ST ALBANS, ST. ALBANS to SAINT ALBANS; PORT ST LUCIE to PORT SAINT LUCIE; ROSLYN HTS to ROSLYN HEIGHTS; JACKSON HGTS to JACKSON HEIGHTS; CAMBRIA HTS to CAMBRIA HEIGHTS; S RICHMOND HL to SOUTH RICHMOND HILL; GARDEN CTY PK to GARDEN CITY PARK; N. PLAINFIELD to NORTH PLAINFIELD; W HEMPSTEAD to WEST HEMPSTEAD; MIAMI BCH to MIAMI BEACH. Units after the change: filings 1,001 cities (1,019 before), owners 799 (821 before); at the City level the page shows 997 (filings, TLC list only) and 684 (owners).

- **ZIP** unit = zip5 as filed. Bubble at the ZCTA internal point from the 2023 ZCTA gazetteer when the ZIP is a ZCTA (`zip_centroids.json`, 1,455 of the 1,553 ZIPs present in either view), else at the mean of the unit's addresses (98 ZIPs: PO-box and unique ZIPs such as 10048, 10101). Label "11230 (Brooklyn, NY)": the commonest city label among every located address of the view in that ZIP (any year, any count option, any group; `zipCities` in each view), so the label is a property of the ZIP and does not move with the measure (pass 8 fix; as first built it was tallied per counted item, so 11363 read Douglaston under two count options and Little Neck under the third).
- **Bubbles**: radius 6 + 34 * sqrt(count / max) px, biggest drawn first so the small ones stay clickable, fill from the ramp by class, the count printed inside when the radius is 14 px or more, sticky tooltip, click = two zoom levels in. Bubble numbers (pass 8 fix): placed biggest bubble first; a number whose box (6.5 px per character plus 2, 13 px tall) would overlap one already placed at the current zoom is left off, and the placement reruns on zoomend. Over NYC at zoom 10: City 13 of 20 candidate numbers drawn, ZIP 34 of 53; at zoom 12 all of them; the headless check measures the drawn boxes and finds no overlapping pair at zoom 10, 11 or 12 in either view. A hidden number is in the ranked table. Still true: hovering the centre of a big bubble that a smaller neighbour overlaps shows the smaller bubble's tooltip (the small one is drawn on top so it stays reachable).
- **Data carried**: owner points get two more indices (21 = city, 22 = ZIP) into shared `cities` ([ST, CITY], 799 entries after the fold fix; 821 as first built) and `zips` (1,152) tables; filings addresses get indices 6 and 7 into their own tables (1,001 cities, 1,019 as first built; 1,322 ZIPs). Blank city: 0 in either view; blank ZIP: 101 owner points, 23 filing addresses (counted under "Outside the US or unknown" at the ZIP level). Data js 9.90 MB (9.58 before the court polygons; 9.22 in pass 7), filings js 11.55 MB (was 11.38).
- **Pinned Address at low zoom**: markercluster with chunkedLoading; at zoom 4 the filings view's 20,324 addresses were in within 3.5 s headless (no fallback to hexagons needed).

### Unit counts (all years, TLC-listed; owners: every address on file, all owner groups)

| Level | Filings, basis B | Owners, every address |
|---|---|---|
| States | 46 | 36 |
| Courts | 80 | 59 |
| Counties | 363 | 195 |
| Cities | 997 (1,015 before the fold fix) | 684 (706 before) |
| ZIPs | 1,308 | 986 |
| Hexagons (0.5 mi, all data) | 3,734 | 2,811 |
| Addresses | 20,324 of 20,395 (TLC-listed, all years) | 28,013 points |

Top five per level, as the ranked table shows them (filings: medallions, every address; medallion-unit pairs in the N line):

| Level | Filings, basis B | Owners, every address on file |
|---|---|---|
| State | New York 13,108; New Jersey 1,351; Florida 337; Pennsylvania 71; Connecticut 52 (15,374 pairs) | New York 10,236; New Jersey 759; Florida 190; Pennsylvania 40; Connecticut 27 (11,393 pairs) |
| Court | E.D.N.Y. 12,103; S.D.N.Y. 5,538; D.N.J. 1,351; S.D. Fla. 229; M.D. Fla. 110 (20,027 pairs) | E.D.N.Y. 8,946; S.D.N.Y. 2,631; D.N.J. 759; S.D. Fla. 124; M.D. Fla. 66 (12,774 pairs) |
| County | Queens 9,745; Kings 5,301; New York 4,446; Nassau 2,416; Bronx 1,255 (27,354 pairs) | Queens 6,402; Kings 2,839; New York 1,935; Nassau 1,247; Bronx 616 (14,968 pairs) |
| City | Brooklyn 5,299; New York 4,437; Long Island City 3,673; Woodside 2,007; Astoria 1,744 (35,835 pairs; 35,859 before the fold fix) | Brooklyn 2,837; New York 1,933; Long Island City 1,891; Woodside 1,056; Astoria 868 (17,841 pairs; 17,863 before) |
| ZIP | 11101 Long Island City 3,635; 11377 Woodside 2,011; 10036 New York 1,193; 10019 New York 878; 11106 Astoria 855 (41,835 pairs) | 11101 Long Island City 1,870; 11377 Woodside 1,063; 10036 New York 451; 11106 Astoria 425; 11230 Brooklyn 364 (20,403 pairs) |

- Headlines as rendered. Filings, landing: "All years, TLC-listed medallions, every address: 93% of 13,154 TLC-listed medallions only ever had a debtor address in New York or New Jersey. 7% (875) had at least one address elsewhere, most often Southern District of Florida (229), Middle District of Florida (110), District of Connecticut (52). Most recent filing: 98% of 13,154 medallions in New York or New Jersey. Filing rows: 98% of 136,532." Owners: "Every address on file: 96% of the 10,461 owners we could map have only ever had New York or New Jersey addresses on their lien filings. 4% (392 owners) have had at least one address elsewhere, most often Southern District of Florida (124), Middle District of Florida (66), District of Connecticut (27)."
- Class breaks at the defaults: state [5, 10, 20, 50, 1000] (six classes), court [2, 5, 10, 20, 50, 1000] (seven), county [1, 2, 5, 10, 200], city [1, 2, 10, 20, 200], ZIP [1, 5, 10, 50, 200], hexagons over NYC [1, 5, 10, 20, 100]; the lowest class (deep blue) reads against the dark basemap at every level (checked on the hexagon view at zoom 10 over NYC, where most hexagons hold 1 to 5).

### Files and pipeline

- New: `_owner_loc_geo_units.py` (city_of, CITY_FOLD, index_table, zcta_points), `261i_build_geo_units.py` (state_shapes.json, zip_centroids.json). Changed: 261f (city and ZIP indices, drivers removed), 261h (city and ZIP indices on addresses), 261g (bundles state shapes and ZIP centroids; inputs stamp), the runner (261i between 261h and 261g), `_owner_loc_panels.py` (drivers() removed), the template and `_owner_loc_page_filings.js` (the view interface: unitCounts, hexItems, legend, note, measureLabel, mixCols). Stage 262 untouched.
- Re-run with `261_run_owner_locations.py --skip-agents`: 261d sent nothing; 261h's three bases equal 262d's summary.json; 261f's latest-point and every-address cross-checks pass; every count on the page is the pass-7 value.

### Checks

- JS parses (six script blocks); no console errors apart from GoatCounter skipping a local file; no sideways page scroll at 1440 or 390 in either view, at any level, with the details open; all three all-years bases equal `summary.json`; GoatCounter head block and the `#me` logic byte-identical to pass 7 (md5 before and after); no em dash in the page, the data or this section; the debtor address is never described as where the owner filed (the lender files the lien).
- Screens: `_screens/pass8/116_pass8_{1440,390}_*.png`: landing (filings, Auto) and the landing answer, State (US) with a real-pointer hover, Court, County, City (NYC) with a bubble tooltip and the ranked table, ZIP (NYC), Hexagon, Address (a real-pointer click opens the popup; Address pinned at zoom 4), the owners answer, owners by City with its table, owners hexagons, the counts tables, the phone control and the phone table, full page.
- Not done at first build (all but the last fixed the same day, below): the copy pass (website-writer); bubble labels overlapping at zoom 10; the Court level shading only the counties that hold a mapped address; the hexagon level has no ranked table (still true).

### Pass 8 fixes (2026-10-12, after the copy pass and two verifiers)

The copy pass (website-writer) ran after the build and rewrote every reader-facing string (title "Medallion lien map", h1 "Where medallions with UCC liens are"). Two verifiers (data; reader and design) then returned findings; every one is applied:

- **Touch: the number stayed up for 200 ms.** On a coarse pointer a tap ends with a synthetic mouseout, and Leaflet's own mouseout handler closed the tooltip the tap had opened. Fix: on touch the page drops that handler (`l.off('mouseout', l.closeTooltip)`) and closes the tooltip itself (on un-highlight, on the second tap before the zoom, on a tap on the map). Chromium touch emulation at 390 (pointer: coarse, with and without the mobile flag): tap on the Brooklyn bubble, tooltip "5,299 / Brooklyn, NY" present at every 100 ms sample over 1.2 s; second tap zooms 10 to 12 and closes it; a tap on Texas at the State level shows "42 / Texas"; a tap on open water clears the tooltip and the highlight. On a phone the tooltip wraps at 220 px instead of running off the map edge.
- **Scroll cue off on the owners view.** The counts-table cue was computed once at build while the owners section was hidden (width 0). Fix: the cue functions are kept and rerun on the next frame after a view switch; after switching to Owners without touching the table, `#countsBox` carries `more` and the hint is on at 1440 (scrollWidth 2,129 against 1,278) and at 390 (1,970 against 324), both tables.
- **Court level patchwork.** Fix: `court_shapes.json` (above); every court is one layer, the NDNY shape covers Hamilton County (no address), and a real-pointer hover there shows "64 / Northern District of New York / Albany, Syracuse, upstate NY" with the court bar and the ranked row lit.
- **Bubble numbers overlapping.** Fix: the greedy placement above, rerun on zoomend; no overlapping pair measured at zoom 10, 11 or 12 on City and ZIP in either view.
- **City fold missed the Census-matched city and common abbreviations.** Fix: the normalisation above runs on both sources; no S OZONE PARK, QUEENS VLG, ST ALBANS, ROSLYN HTS, CAMBRIA HTS, W HEMPSTEAD, JACKSON HGTS, LONGISLANDCITY, QUEENSVILLAGE, PORT ST LUCIE or S RICHMOND HL unit is left (South Ozone Park 250 medallions, Saint Albans 57, Queens Village 485, Roslyn Heights 68, West Hempstead 45, Port Saint Lucie 10 on the landing measure).
- **ZIP label moved with the count option.** Fix: the label names the commonest city among every address of the view in that ZIP (above); 11101, 11363, 10036, 33313 and 44514 read the same under all three count options (11363 Little Neck, NY; 33313 Lauderhill, FL; 44514 Youngstown, OH; 33024 Hollywood, FL, which has no unit under the latest-filing option).
- **FIGURES.md** 116 row said the copy pass was pending; it now says done.
- **Second wording pass (website-writer, 2026-10-12).** The live wording is in the page; the title and headlines quoted in the pass-8 record above are the builder's drafts. Court codes in the address-rule line are spelled out, the bar summaries and legend hints are shorter, and the ranked table hides its click line when it has no rows.
- Noted, no change: no mapped point lacks a county in either view, so the "address with no county takes its state code" branch never fires (the 107 owner points without a county are the unmapped ones, not drawn); the Court level's "No US court found" unit has no shape, so its ranked row cannot zoom or highlight; the build stamp in the figure folder's html differs from the clone's until the clone is refreshed (done at the end of this pass).
- Reader-facing strings added or changed by this pass, for the website-writer: the Court mode note ("Each bankruptcy court is shaded by its number of medallions, as one shape." and the owners twin), the Details "Group by" bullets for Court, City (the abbreviations) and the new "Bubble numbers" bullet, and the ZIP label bullet.
- Checks after the fixes: runner `261_run_owner_locations.py --skip-agents` exit 0 (261d "to send 0"; 261e "court shapes 91 districts, 172 rings, 16315 points"; 261f latest-point and every-address cross-checks pass; 261h three bases equal 262d's summary.json); all four inline script blocks parse (new Function); headless Edge at 1440 and 390: landing on Medallions with liens, basis Medallions every address, Group by Auto; all three bases match summary.json; no sideways scroll at either width in either view at any level or with Details open; no placeholder cell; no "n/a", "hack licences", em dash or "filed from" in the page text; the only console error is the GoatCounter script blocked by the test harness; GoatCounter head block and the #me toast byte-identical to the clone's HEAD (md5 6be8c1fe... and 1dc302bb...). Every count on the page is the pass 8 value except the City level (997 and 684 cities, pairs 35,835 and 17,841).
- Screens: `_screens/pass8/116_pass8v_{1440,390}_*.png` (landing, State with a real-pointer hover, Court with a hover on a county that holds no address, County, City, ZIP, Hexagon, Address, owners City and Court, the counts tables, the phone control and phone table, the phone tap), plus `_tops_v_{1440,390}.json`.

## Pass 9 (2026-10-13): unmatched addresses placed by ZIP and city, City units from ZIP, owners boxes removed, matching explained

William's messages, 2026-10-12 and 2026-10-13, verbatim: "i understand that some places arent able to match with the census for those ones can you just try to extract them from zipcode and city , etc"; "take this out" (the owners view's summary box, "Where the owners are"); "can you add an explantion of the matching so we have differnt type of matching yes ? the first set of matching is matched from the TLC file to our UCC do we hae the full set of TLC files from 2013 to toay. the second is straight just lookign at all the medalliosnrealted filing in the UCC where a medallion is named and plotting those locations"; ".... take out htis portion" (the owners view's Details box); "when you plotted medallions with liens you made sure to also include inactive liens yes?" (yes; the explanation block says so). Copy on the page is the builder's draft; a website-writer pass rewrites it.

### What changed

- **Addresses the Census could not place are now on the map.** A new module, `_owner_loc_fallback.py`, builds a pool of every Census-matched address on both sides (41,445 addresses: 1,849 ZIPs, 1,283 city units; 00000 is no ZIP, and a ZIP is a key only with 3 or more pool addresses, both since the fix below) and tries five routes in order. Precision "ZIP-level" (route 1) or "city-level" (routes 2 to 5); the page calls the new tier "city area only" and draws it as a dashed ring.
- **City units of unmatched addresses are checked against the ZIP.** `city_unit`: a Census-matched address keeps its Census city; an unmatched address keeps its filed city (folded) when the Census found that city in the address's ZIP (11101 holds Long Island City, Astoria and Sunnyside), else takes the ZIP's commonest Census city. The check needs 3 or more pool addresses in the ZIP; a thinner ZIP, or 00000, leaves the filed city. Both views, the ranked table and the ZIP labels rest on it.
- **Owners view, two boxes out** (William): the summary box ("Where the owners are", the two headline sentences, the "392 owners (4%)" list and its six bullets) and the whole "Details: owners by name" box, with the JS that filled them. Nothing from them is recreated elsewhere. The view keeps the switch, the Bankruptcy courts bars with the group chips and the Count control, the map and Group by, the ranked table, the side panels, the Find a name search and both Counts by group tables.
- **Explanation block "Two ways onto the map"**, a panel directly under the view switch, visible in both views (text below).
- Stage 262 untouched (md5 of every file under `data/2_analysis/medallion_filing_locations/` identical before and after); the geocoder caches untouched; 261d sent 0 addresses.

### Routes, in order

| Route | Point | County | Precision | Label on the data |
|---|---|---|---|---|
| 1 | median of the pool's addresses with the same ZIP (3 or more of them; 00000 is no ZIP) | their modal county | ZIP-level | ZIP, middle of our matched addresses |
| 2 | median of the pool's addresses in the same city unit (3 or more) | modal county | city-level | city, middle of our matched addresses |
| 3 | Census 2023 place gazetteer internal point (same state, same name after the type word is removed; a non-CDP first, then the largest land area) | point in polygon (261e / 261h) | city-level | city, Census place point |
| 4 | as route 2 with 1 or 2 pool addresses | modal county | city-level | city, middle of our matched addresses (fewer than 3) |
| 5 | the filed city equals a county name of the state (QUEENS, NY): median of the pool's addresses in that county | that county | city-level | county name, middle of our matched addresses |
| 6 | none | | unmapped | stays off the map |

Owners side (261e, written to `addresses_located.csv`, new column `fallback`):

| | Addresses |
|---|---|
| Unmapped before | 153 |
| Route 1, ZIP | 9 |
| Route 2, city (3 or more) | 104 |
| Route 3, Census place point | 25 |
| Route 4, city (fewer than 3) | 4 |
| Route 5, county name (QUEENS, NY) | 1 |
| Still unmapped | 10 |

Still unmapped, owners (city, state, ZIP as filed): MELVILLE NE (2 addresses); NEW YORK NE; NEW YO NY (437 Madison Ave); CLARK NY; HOLLISWOOD NY; NORTH WOODMERE NY; LINCOLN NY 13043; PLAINVILLE NY 13137; LIONO LAKES MI 55014. HOLLISWOOD and NORTH WOODMERE are neighbourhood names: the 2023 place gazetteer has no entry for either (only Woodmere CDP), and the pool keys its matched addresses by the Census city, so a neighbourhood filed as the city has no unit to land in. No fold was invented for them.

Filings side (261h, in memory; nothing under `medallion_filing_locations/` written):

| | Addresses | Filing rows | Medallions |
|---|---|---|---|
| Unmapped before | 106 | 150 | 146 medallion-rows |
| Route 1, ZIP | 1 | | |
| Route 2, city (3 or more) | 92 | | |
| Route 3, Census place point | 5 | | |
| Route 4, city (fewer than 3) | 1 | | |
| Recovered | 99 | 143 | 130 (all TLC-listed; none new to the map, each already had a located row) |
| Still unmapped | 7 | 7 | 8 |

Still unmapped, filings: HOLLISWOOD NY (2 addresses), CRONE PARK NY, LEFRAK CITY NY, LINCOLN NY 13043, PLAINVILLE NY 13137, "1310" PA (the street number filed as the city). Of the 8,790 unlocated rows with no guard, 8,640 carry no address at all and are not recoverable.

Precision after the pass:

| Tier | Owners (24,891 addresses) | Filings on the page (20,494 addresses) |
|---|---|---|
| exact | 18,356 | 16,987 |
| approximate | 4,532 | 1,550 |
| ZIP area only | 1,859 (1,850 before) | 1,859 (1,858 before) |
| city area only | 134 | 98 |
| unmapped | 10 (153 before) | 7 of 106 before (the rest of 262's 106 sit on rows the page does not draw) |

Owner points drawn: 28,110 (28,013 in pass 8). Filing rows on the page: 137,181 (137,038); addresses 20,494 (20,395).

### City units: moves old to new

On the addresses each view draws, `city_unit` differs from pass 8's `city_of` for 105 owner addresses (89 distinct old-to-new pairs) and 26 filing addresses (24 pairs). All are unmatched addresses whose filed city is not one the Census found in their ZIP, the ZIP holding 3 or more matched addresses. The ten commonest moves, both sides together:

| Old unit | New unit | Addresses |
|---|---|---|
| Bronx, NY | New York, NY | 4 |
| Jackson Heights, NY | East Elmhurst, NY | 4 |
| Queens, NY | Jamaica, NY | 4 |
| Elmhurst, NY | East Elmhurst, NY | 3 |
| Jamaica, NY | Queens Village, NY | 3 |
| Melville, NY | Garden City, NY | 3 |
| Queens, NY | Woodside, NY | 3 |
| Old Brookville, NY | Greenvale, NY | 2 |
| Queens, NY | Ozone Park, NY | 2 |
| S.Richmond Hill, NY | South Richmond Hill, NY | 2 |

The rule follows the ZIP even when the filed city is a real place (Melville, NY with a Garden City ZIP; Kew Gardens with a Brooklyn ZIP): the ZIP is the geocoded key and the filed city was not among the ZIP's Census cities. City units: owners 777 (799 in pass 8), filings 995 (1,001); on the page 663 (684) and 991 (997).

Folds added (filed-city path only): B'KLYN, JAMACIA, BELLROSE, RICHMONDHILLS, AMITVILLE, GREEN PORT, WEST HAMPTON BEACH; and the neighbourhood table CONEY ISLAND to BROOKLYN, RIVERDALE to BRONX (NY only, never applied to a Census city). Unmatched addresses they catch: owners BELLROSE 5, WEST HAMPTON BEACH 3, JAMACIA 2, B'KLYN 2, RICHMONDHILLS 1, GREEN PORT 1, AMITVILLE 1, RIVERDALE 1; filings B'KLYN 1, WEST HAMPTON BEACH 1, CONEY ISLAND 1.

### Headline numbers, before and after

- Filings view, landing (Medallions, every address, all years, TLC list only): "93% of 13,154 medallions with a UCC lien have only ever had a debtor address in New York or New Jersey. 98% have their latest lien filing there." Unchanged at the rounding shown. Details, NY or NJ share: medallions in every court 95.3% (95.4% in 262d's summary), latest filing 98.0% (98.0%), lien filings 98.3% (98.3%).
- Owners view (box removed; the bar summary line): 10,461 owners; 10,068 (96.2%) appear only in New York or New Jersey courts, 393 (3.8%) elsewhere as well (392 in pass 8).

Top five per level, as the ranked table shows them (pass 8 value in parentheses where it moved):

| Level | Filings, medallions every address | Owners, every address on file |
|---|---|---|
| State | New York 13,109 (13,108); New Jersey 1,351; Florida 339 (337); Pennsylvania 73 (71); Connecticut 52 (15,386 pairs; 15,374) | New York 10,237 (10,236); New Jersey 759; Florida 190; Pennsylvania 40; Connecticut 27 (11,395 pairs; 11,393) |
| Court | E.D.N.Y. 12,103; S.D.N.Y. 5,551 (5,538); D.N.J. 1,351; S.D. Fla. 229; M.D. Fla. 112 (110) (20,051 pairs; 20,027) | E.D.N.Y. 8,946; S.D.N.Y. 2,637 (2,631); D.N.J. 759; S.D. Fla. 124; M.D. Fla. 66 (12,781 pairs; 12,774) |
| County | Queens 9,745; Kings 5,305 (5,301); New York 4,459 (4,446); Nassau 2,416; Bronx 1,255 (27,383 pairs; 27,354) | Queens 6,402; Kings 2,839; New York 1,941 (1,935); Nassau 1,247; Bronx 616 (14,975 pairs; 14,968) |
| City | Brooklyn 5,303 (5,299); New York 4,451 (4,437); Long Island City 3,674 (3,673); Woodside 2,007; Astoria 1,746 (1,744) (35,856 pairs; 35,835; 991 cities, 997) | Brooklyn 2,838 (2,837); New York 1,941 (1,933); Long Island City 1,891; Woodside 1,056; Astoria 870 (868) (17,832 pairs; 17,841; 663 cities, 684) |
| ZIP | 11101 Long Island City 3,635; 11377 Woodside 2,011; 10036 New York 1,193; 10019 New York 878; 11106 Astoria 855 (41,967 pairs; 41,835; 1,339 ZIPs, 1,308) | 11101 Long Island City 1,870; 11377 Woodside 1,063; 10036 New York 451; 11106 Astoria 425; 11230 Brooklyn 364 (20,452 pairs; 20,403; 1,006 ZIPs, 986) |

Units per level: states 46 / 36; courts 80 / 59; counties 366 (363) / 195; cities 991 / 663; ZIPs 1,339 / 1,006.

### The three bases over all rows, against 262d

261h still asserts the three bases on the rows 262 located equal `summary.json` (they do). Over all rows on the page (the 143 recovered rows included), TLC-listed medallions:

| Base | SDNY | EDNY | NDNY | WDNY | D.N.J. | Other | 262d's (SDNY, EDNY, other) |
|---|---|---|---|---|---|---|---|
| A, lien filings | 22,986 | 104,015 | 114 | 49 | 7,219 | 2,292 | 22,938, 103,937, 2,275 |
| B, medallions every address | 5,551 | 12,103 | 64 | 43 | 1,351 | 939 | 5,538, 12,103, 928 |
| C, medallions latest filing | 2,488 | 9,834 | 10 | 7 | 548 | 267 | 2,487, 9,835, 267 |

Every medallion (TLC setting off): A 23,048 / 104,288 / 114 / 50 / 7,263 / 2,418 (262d: 23,000 / 104,210 / 114 / 50 / 7,263 / 2,401); B 5,678 / 12,502 / 64 / 44 / 1,390 / 1,016 (5,665 / 12,502 / 64 / 44 / 1,390 / 1,005); C 2,607 / 10,227 / 10 / 8 / 583 / 341 (2,606 / 10,228 / 10 / 8 / 583 / 341). The page's live bases equal 261h's `counts.bases_all` on all three (headless check); the Details "Three ways to count" line now quotes the all-rows shares.

### Sources

- US Census Bureau 2023 Gazetteer Files, places, downloaded 2026-10-12: `data/0_raw/census_geo_2023/place_gaz/2023_Gaz_place_national.txt` (32,329 places; 32,104 distinct state and name keys after the type word is removed). Not under `gaz/`, which 261d globs.
- The pool: `owner_locations/addresses_geocoded.csv` (261d) and `medallion_filing_locations/filing_addresses.csv` (stage 262, read-only).
- County names for route 5: `cb_2023_us_county_500k`.

### The explanation block (page text, builder's draft)

Heading "Two ways onto the map", one bullet per view, under the view switch in both views:

- Medallions with liens. Every UCC (Uniform Commercial Code) filing in our LexisNexis collection whose collateral names a medallion number, placed at the debtor address on the filing. No name matching. Filings 1966 to 2026. Open, terminated and lapsed liens all count. 137,181 located filings naming 13,154 medallions on the NYC Taxi and Limousine Commission (TLC) lists. The collection covers 95.7% of the medallions we set out to collect.
- Owners by name. The TLC owner lists, everyone who held a medallion on the TLC record from 7 March 2013 to early 2026 (the TLC published no owner list before 2013), matched by name to the debtor names on UCC filings, word order ignored. On the LexisNexis route the filing must also name one of that owner's own medallions. 14,227 entries on the lists; 10,461 have an address on the map.

The two counts in the first bullet and the two in the second are filled from the data (`counts.rows`, `counts.medallions_tlc`; `names_on_list`, `names_mapped`); the 95.7% and the dates are literal. Wording fix 2026-10-13: LexisNexis, UCC and the NYC Taxi and Limousine Commission spelled out on first use in the block; "medallion worklist" replaced by "medallions we set out to collect".

### Page changes

- Legend (both views' "At address level" panels): a fourth tier, "city area only", a hollow ring with a dashed outline (Leaflet circleMarker `dashArray: '3 3'`; the key draws it with `stroke-dasharray`). Popups: "Location match: city area only". The owners side panel carries one line under the key with the rule and the counts (134 addresses as city area only; 10 stay off the map). The filings Details "Geocoding" section lists the Census result, the 99 placed and the 7 off the map, and the tiers on the map; "What the data holds" says 137,181 placed on the map, 143 of them by ZIP or city.
- Details "Group by", City: the neighbourhood folds and a new bullet on the ZIP check. The City mode note in both views says "checked against the ZIP".
- Removed: the owners summary panel (`#answer`, `#answer2`, `#facts`) and the owners Details (`#method`, `#methodBody`) with their JS; the footer line stays. The missing-data guard now writes into the filings headline.

### Files and pipeline

- New: `_owner_loc_fallback.py` (Pool, `place_unmapped`, `city_unit`, `ROUTES`). Changed: `_owner_loc_geo_units.py` (seven fold entries, `NYC_NEIGHBOURHOOD`, docstring), `261e_assign_districts.py` (fallback before the county step; `fallback` column; route tally printed), `261f_build_owner_location_points.py` (PREC_RANK gains city-level and unmapped 4; `city_unit`; geocode summary carries the tiers and the routes), `261h_build_filing_layer.py` (`recover`; the 262d cross-check on the rows 262 located; `counts.recovered`, `counts.precision`, `counts.bases_all`), the runner's docstring, the template and `_owner_loc_page_filings.js`. 261d and stage 262 untouched.
- Re-run with `261_run_owner_locations.py --skip-agents`, exit 0: 261d "to send 0"; 261e route tally as tabulated; 261f cross-checks pass; 261h's assert passes and prints the recovered counts and the all-rows bases.

### Checks

- All four inline script blocks parse (new Function); headless Edge at 1440 and 390: no console error apart from GoatCounter declining localhost; no sideways scroll on the landing, the City level, the Details open or the owners view; the three bases on the page equal 261h's `bases_all`; the city-area popup shows the tier line; no element of the two removed boxes remains (`#answer`, `#facts`, `#method`, `#methodBody`, `#h-answer` absent); the owners view shows Two ways onto the map, Bankruptcy courts, Map and Counts by group (4 and 3 rows).
- md5 of every file under `medallion_filing_locations/` identical to the pre-pass record; `geocode_cache.csv` identical; no em dash in any changed module, the page, the data files or this section, and the debtor address is nowhere described as where the owner filed; GoatCounter head block and the `#me` toast byte-identical to the Pages clone (md5 b43a53cb, 050d3d76).
- Screens: `_screens/pass9/116_pass9_{1440,390}_*.png`: the landing with the explanation block, the block alone, the four-tier legend (filings and owners panels), a city-area popup (114-38 Jewel Ave, Flushing, NY; Eastern District of New York; no ZIP), the owners view top and its Counts tables and full page, the City ranked table over NYC, the Details with the geocoder sentences; `_tops9.json` holds the ranked tops.
- Not done: the website-writer copy pass on the new strings (the explanation block, the tier note, the Details lines). A filed neighbourhood name (Holliswood, North Woodmere, Lefrak City, Crone Park) has no route; a pool keyed by the filed city of matched addresses would catch the first two and is not built.

### Pass 9 fix (2026-10-13, after the data verifier's hand check)

The verifier hand-checked all 242 recovered addresses and found three wrong courts, two wrong counties and one wrong City unit, all from the ZIP route and the City-unit ZIP check accepting any ZIP in the pool: the placeholder 00000 (43 matched owner addresses, spread over New York, Kings, Queens and Nassau counties, median point in Woodside) and ZIPs with 1 or 2 pool addresses (1,138 of 1,850). Fix in `_owner_loc_fallback.py`: 00000 is no ZIP (`NO_ZIP`), and a ZIP is a key only with 3 or more pool addresses (`ZIP_MIN`, `Pool.zip_ok`), for route 1 and the City-unit check alike; otherwise the address takes the city route and keeps its filed city. The tables and counts above are restated to the fixed run.

- Routes before and after. Owners: ZIP 26 to 9; city 90 to 104; Census place point 24 to 25; city with fewer than 3 addresses 2 to 4; county name 1; still unmapped 10, the same ten. Filings: ZIP 8 to 1; city 85 to 92; place point 5; fewer than 3 1; still unmapped 7, the same seven. Recovered filing rows, addresses and medallions unchanged (143, 99, 130).
- The hand-checked addresses now: 149 Linden Ave, Irvington NY 00000 to Westchester, SDNY (was Queens, EDNY); 210 E 181st St, New York NY 11047 to New York County, SDNY (was Nassau, EDNY, unit Manhasset Hills; the court is right, the county is the city route's New York rather than the street's Bronx); 82-24 Bay Parkway, Brooklyn 00000 to Kings (was Queens); 399 Leversell St, Staten Island 11308 to Richmond, unit Staten Island (was Queens, Corona); PO Box 863741, Ridgewood 11386 keeps unit Ridgewood (was Corona).
- Not fixed by this rule: 225 Broadhollow Road #132W, Melville NE 10001 still sits in New York County (SDNY) by its ZIP; 10001 holds 145 pool addresses, so the minimum does not reach it. A filed-state check would catch it, but it would also unmap two shifted-field filings (35-46 Crescent St and 91-39 11th Street, state filed as AL, ZIPs 11102 and 11418) that the ZIP places correctly in Queens. Left as is.
- Moved on the page: owners ZIP area only 1,876 to 1,859 and city area only 117 to 134; filings 1,866 to 1,859 and 91 to 98. City-unit moves on drawn addresses 113 to 105 (owners) and 29 to 26 (filings); the top-ten table above is unchanged. City units on the page: owners 663 (was 662). Base A, lien filings, TLC-listed: SDNY 22,982 to 22,986, EDNY 104,019 to 104,015 (the rows at 210 E 181st St); B and C unchanged. Ranked tops: filings County Queens 9,746 to 9,745, New York 4,458 to 4,459, Nassau 2,421 to 2,416; City New York 4,450 to 4,451; owners County Queens 6,404 to 6,402; City Brooklyn 2,839 to 2,838. Headlines unchanged (93% / 98%; owners 96.2%, 393 elsewhere).
- Page text: the Group by City bullet gains "The check needs 3 or more of our matched addresses in the ZIP; 00000 is not a ZIP"; the owners legend note and the filings Details Geocoding line read "in the same ZIP (3 or more of them)". Docstrings of the module, 261e, 261f and 261h updated.
- Checks after the fix: runner `--skip-agents` exit 0 (261d "to send 0"; 261f cross-checks and 261h's assert pass); stage 262's 13 files and `geocode_cache.csv` md5-identical before and after; all four inline script blocks and both data files parse (new Function); headless Edge at 1440 and 390: the three bases equal `bases_all`, no sideways scroll, no console message beyond GoatCounter declining localhost, the removed owners elements absent, the city-area popup intact; GoatCounter head block and the `#me` toast byte-identical to the clone's HEAD; no em dash, and the debtor address is nowhere described as where the owner filed.
- Screens: `_screens/pass9/fix/116_pass9_{1440,390}_*.png` (the same twenty views as the pass 9 set) and `_tops9.json`.

## Pass 10 (2026-10-13): filings boxes removed, continuous blue-to-red ramp

William's messages, 2026-10-13, verbatim: "yes take this out" and "make sure to take this out please" (the filings view's summary box, "Where the medallions are"); "tahe this out as well" (the filings view's Details box, "Details: medallions with liens"); "for hte visualizaiton can you put a more cintounce colloring is that possible not discrete ocnitnous"; "also the color scheme is confusing it should go from blue cold to red hot why is orange at the ennd of the scale fix this please get the websuit deisgn agent on this thisis bad ui design no?".

### What was removed

- **Filings view, summary box:** the "Where the medallions are" section, its two headline sentences, the list under them and every facts bullet, with the JS that filled them. Nothing from it is recreated elsewhere.
- **Filings view, Details box:** "Details: medallions with liens" with all of its sections (What the data holds, Three ways to count, Group by, Left out, Geocoding, Map and search), with the JS that filled it and the Group by list it printed. Nothing from it is recreated elsewhere.
- **Kept:** the title and subtitle, the view switch, the "Two ways onto the map" block, the Count, years and TLC controls, the court bars, the map and Group by, the legends, the ranked table, the medallion history search. The missing-data message now shows in the filings line of the "Two ways onto the map" block.
- Page only: stage 262, the 261 data builders and every data file are untouched (md5 of both JSON files and every file under `data/2_analysis/medallion_filing_locations/` identical before and after; the built `_data.js` and `_filings.js` byte-identical to pass 9).

### Count and Measure buttons

The Details box explained the count options, so each button now carries a one-line explanation, shown on hover and keyboard focus (and read out as the button's description). A page tooltip replaces the browser's title tooltip, which waits a second and does not show on touch.

| Button | Text |
|---|---|
| Medallions, every address | A medallion counts once in every court it ever had a debtor address in |
| Medallions, latest filing | Each medallion counts once, at the debtor address on its newest filing |
| Lien filings | Every placed lien filing counts once |
| Every address on file (owners) | A name counts once in every court where it has an address, any year |
| Most recent address (owners) | The address on the name's newest filing of any kind |

The fourth location tier's rule (the Census could not place the address) now sits under the tier key in both views: `#precNote` in the owners view, `#fprecNote` in the filings view, same sentence, each with its own counts (filings: 98 city area only, 7 off the map).

### The ramp

- **One continuous ramp** replaces the seven quantile classes. Stops, evenly spaced, in one constant (`RAMP_STOPS` in the template): `#8cc0ff` light blue, `#9a90f4` violet, `#c25ccc` magenta, `#da3a66` rose, `#c71f2a` red. Nothing follows red (the old top class was orange). Colours between stops are straight RGB blends.
- **Log scale.** Colour is log10(count) over log10(the level's largest count): 1 is blue, the largest is red. The domain is recomputed whenever the level, view, years, count option or TLC setting changes. Counts run 1 to about 13,000, so a linear scale would paint nearly every unit blue.
- **Same code for every level:** state, court and county polygons, city and ZIP bubbles, hexagons, in both views. A zero count keeps the no-data fill (no fill).
- **Legend:** one gradient bar with the whole ramp; ticks at 1, 10, 100, 1,000 and 10,000 where they fall inside the domain, plus the largest count. A power of ten within 30% of the bar of the largest count is left off so labels do not touch (state and court: 1, 10, 100 and the maximum; county, city, ZIP and hexagon the same). The bar is 216 px wide on a desktop and 168 px on a phone.
- **Checked:** lightness falls steadily from the light-blue end to the red end under normal vision, deuteranopia and protanopia (Machado simulation, 17 points along the bar, relative luminance). The earlier saturated-blue ramp lost order in the middle under deuteranopia, so the low end was lightened and the top darkened.
- Largest count per level, TLC list, every address, all years: state 13,109; court 12,103; county 9,745; city 5,303; ZIP 3,635; hexagon 1,454. Owners view, every address on file: state 10,237; court 8,946; county 6,402; city 2,838; ZIP 1,870; hexagon 769.
- Hover highlight, tooltip with the exact count and table row lighting are unchanged. The class-break code (`breaks`, `classOf`, `rampFor`, `scaleRows`) is gone.

### Checks (headless, Edge, 1440 and 390)

- No console errors beyond the GoatCounter script (blocked on purpose); no horizontal scroll at 320, 390 or 1440.
- Legend ticks do not overlap and sit inside the legend at every level in both views, at both widths.
- Distinct fill colours per level, filings view 1440: state 29, court 28, county 40, city 96, ZIP 133, hexagon 138.
- No em dash in the template, the filings module, the built page or this note.

### Screens

`_screens/pass10/116_pass10_{1440,390}_{n}_{what}.png`: landing, state, court, county, city, ZIP, hexagon, legend close-up, page top, owners landing and owners map; 1440 also the Count button tooltip.

## Status

Built 2026-10-12, passes 2 and 3 the same day; re-run the same day with the DOS medallion-lender rule (pass 3), the own-medallion rule (pass 4), the redesign (pass 5), the verification fixes (pass 6), the two measures, people split, folded places, guards and loan-filings view (pass 7), and the Group-by levels, filings landing view, every-address default, blue-to-red ramp, hover highlight and the drivers box removal (pass 8). Published on GitHub Pages at William's request; the pass-8 files (copy pass and fixes included) are copied into the Pages clone, not committed. Passes 4 to 8 not committed or pushed. Nothing sent to Ed. The pass-8 copy pass (website-writer) and the fix pass after the verifiers are done.
- A stale pass-1 copy (names with addresses) sits outside the project at `~/Sites/taxi-map/` (synced by `~/Sites/taxi-map-bin/sync_taxi_map.sh`), served by `python -m http.server 8734` on 127.0.0.1 and proxied tailnet-only by Tailscale at `/taxi-map`. Not created by stage 261; left as is.
