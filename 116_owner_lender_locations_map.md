# Figure 116: where the owners, lenders and agents in our data filed from (interactive map, LOCAL ONLY)

## Goal

- Ed, 2026-10-12: "Do all taxi drivers/companies live in New York and New Jersey? I need to know their locations in order to request bankruptcy data from the appropriate court."
- Map every owner, lender and TLC managing agent we hold an address for, and give each one's federal (bankruptcy) district.

## Output

- `116_owner_lender_locations_map.html`: open locally. Leaflet 1.9.4 and markercluster 1.5.3 are inlined. Data is in the sibling `116_owner_lender_locations_map_data.js` (6.7 MB).
- Six groups, each with its own colour AND shape, used the same way on the map, in the legend, in popups and in the counts table:

| Group | Shape | Colour | Points | Names mapped |
|---|---|---|---|---|
| Individual owners | circle | #0072B2 blue | 23,286 | 4,597 |
| Companies (LLC) | square | #E69F00 orange | 2,617 | 1,615 |
| Companies (other: INC, CORP, etc.) | diamond | #CC79A7 pink | 15,579 | 4,120 |
| Lenders | triangle | #009E73 green | 1,337 | 578 |
| Lender-held series LLCs (730 series + 2 DePalma acquisition vehicles) | down triangle | #D55E00 vermilion | 732 | 732 |
| TLC managing agents | star (white halo, larger) | #1A1A1A black | 95 | 82 |
| TLC drivers | no map element | | 0 | 0 |
| Crash drivers | no map element | | 0 | 0 |

- Colours: Okabe-Ito. CVD check (Machado simulation, CIEDE2000): smallest pair distance 12.2 protan, 12.5 deutan, 10.9 tritan, 22.2 normal. Markers carry a darker outline, since orange on the grey basemap is only 1.9:1.
- Markers are canvas paths (no DOM per point). 43,646 points load in about 5 s headless; a layer toggle takes 0.1 to 0.3 s.
- Cluster bubbles: one cluster layer for all groups; each bubble is a donut whose ring shows the group mix.
- Hover a bubble: per-group counts with each group's shape (readable without colour).
- Cluster radius shrinks with zoom (44 px, 20 px from zoom 15, 6 px from zoom 17); same-address stacks still cluster and spiderfy.
- Toggles: each group; owner points from DOS name search only (17,617); owner points at a managing-agent office (6,815); district lines. The counts table switches between All sources and Lexis-backed only.
- Popups tag: "At a managing-agent office", "Also a TLC managing agent" (11 lenders), "Medallion-number ACQUISITION LLC", and the lender's medallion-test result.
- Pass 3 (2026-10-12) screens: `116_desktop_1440_lic16/17/18.png` (Woodside at street zoom), `_lic17_grey.png` (greyscale), `_cluster_tooltip.png`, `_agent_star.png`, `_group_rules.png`; pass-2 renders moved to `_screens/pass2/`.
- `_screens/`: 1440 and 390 px (full page, viewport, counts both bases, layers, drivers, notes, cluster view, popups) and one zoomed crop per shape (`116_desktop_1440_shape_*.png`). No console errors; no sideways scroll at either width. Pass-1 screenshots moved to `_screens/pass1/`.
- **LOCAL ONLY: the page holds names with addresses. Do not publish or upload it.**

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

## Counts (stage 261f, pass 2)

District columns count each name once, at its latest mapped address that is not a managing-agent office (else its latest address). "Any address outside NY/NJ" counts names with at least one mapped address outside SDNY, EDNY, NDNY, WDNY and D.N.J.

All sources:

| Group | On list | Mapped | Addresses | SDNY | EDNY | NDNY | WDNY | D.N.J. | Other | Any address outside NY/NJ |
|---|---|---|---|---|---|---|---|---|---|---|
| Individual owners | 5,117 | 4,597 | 17,477 | 498 | 3,517 | 20 | 18 | 328 | 216 | 428 |
| Companies (LLC) | 3,952 | 1,615 | 1,577 | 340 | 1,192 | 1 | 2 | 71 | 9 | 11 |
| Companies (other) | 5,158 | 4,120 | 5,963 | 973 | 2,949 | 2 | 3 | 141 | 52 | 69 |
| **All owners** | 14,227 | 10,332 | 23,498 | 1,811 | 7,658 | 23 | 23 | 540 | 277 | 508 |
| Lenders | n/a | 578 | 1,084 | 160 | 242 | 12 | 3 | 50 | 111 | 122 |
| Lender-held series LLCs (730 series + 2 DePalma) | n/a | 732 | 17 | 478 | 254 | 0 | 0 | 0 | 0 | 0 |
| TLC managing agents | n/a | 82 | 78 | 17 | 65 | 0 | 0 | 0 | 0 | 0 |
| TLC drivers | 56,648 | 0 | 0 | | | | | | | |
| Crash drivers | n/a | 0 | 0 | | | | | | | |

Lexis-backed only:

| Group | On list | Mapped | Addresses | SDNY | EDNY | NDNY | WDNY | D.N.J. | Other | Any address outside NY/NJ |
|---|---|---|---|---|---|---|---|---|---|---|
| Individual owners | 5,117 | 4,596 | 9,619 | 486 | 3,543 | 6 | 4 | 344 | 213 | 314 |
| Companies (LLC) | 3,952 | 1,615 | 1,139 | 336 | 1,197 | 1 | 0 | 73 | 8 | 10 |
| Companies (other) | 5,158 | 4,120 | 3,652 | 965 | 2,958 | 2 | 2 | 139 | 54 | 67 |
| **All owners** | 14,227 | 10,331 | 13,458 | 1,787 | 7,698 | 9 | 6 | 556 | 275 | 391 |
| Lenders | n/a | 434 | 745 | 132 | 153 | 9 | 3 | 43 | 94 | 102 |
| Lender-held series LLCs (730 series + 2 DePalma) | n/a | 732 | 17 | 478 | 254 | 0 | 0 | 0 | 0 | 0 |
| TLC managing agents | n/a | 82 | 78 | 17 | 65 | 0 | 0 | 0 | 0 | 0 |

- **Owners: 10,055 of the 10,332 mapped (97.3%) have their latest non-agent address in NY or NJ** (pass 1, latest address of any kind: 10,111, 97.9%).
  - The other 277: S.D. Fla. 98, M.D. Fla. 53, D. Conn. 16, E.D. Pa. 12, M.D. Pa. 12, C.D. Cal. 10, N.D. Ga. 10, E.D. Cal. 6, and 60 more in 35 other districts.
  - Lexis-backed only: 10,056 of 10,331 (97.3%).
  - 22 of the 277 rest only on DOS name-search rows.
- DOS v1 and v2 owner rows are name-only links: 17,617 of 41,482 owner points (42.5%) are DOS-only and may be a same-name stranger.
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

## Details

- **Whose address.** Owner addresses are the UCC debtor address at the filing date; median filing year 2008. Often a managing agent or garage, not a home; not current.
- **Owner linking.** Lexis: collateral names the owner's own medallion AND `name_match.match` is exact, set or subset. DOS v1 and v2: name match plus `ucc_lien_matched`; the medallion column is the searched owner's, so name-only.
- **Lexis-backed basis.** Lexis records, DOS rows at an address the same owner has on Lexis, DOS corporations, TLC.
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

## Pass 3 (2026-10-12): what changed

- Precision key drawn with a neutral circle (the diamond is the company_other shape).
- Agent star: radius 7 to 8.5, white outline halo (ZIP-level stars keep the dark outline); spider legs light grey.
- Cluster radius shrinks with zoom; bubble hover lists per-group counts with shapes.
- Series LLC 732 split on the page as 730 series + 2 DePalma (stage 261b `spv_series`, `spv_no_medallion`).
- Counts unchanged (all rows identical to pass 2).

## Status

Built 2026-10-12, passes 2 and 3 the same day. Not committed (the page holds names with addresses; the repo has a GitHub remote). Nothing sent to Ed.
- A stale pass-1 copy (names with addresses) sits outside the project at `~/Sites/taxi-map/` (synced by `~/Sites/taxi-map-bin/sync_taxi_map.sh`), served by `python -m http.server 8734` on 127.0.0.1 and proxied tailnet-only by Tailscale at `/taxi-map`. Not created by stage 261; left as is.
