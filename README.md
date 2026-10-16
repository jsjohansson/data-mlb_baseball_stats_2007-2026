# 20 Years of Modern Baseball

Every table Baseball-Reference publishes for all 30 Major League clubs across the 20 seasons
from 2007 to 2026: **108 CSV tables, 2,655,896 rows, 3,767 columns**, every column defined.

Batting, pitching and fielding at both standard and advanced level; value and WAR components;
standings, schedules and results; rosters, coaching staffs and transactions; award voting;
postseason series and box scores; and game-level batting orders and defensive alignments.

**Don't want to use CSV -- No Problem -- Recommend you use these alternatives over the CSV package**

Data download of a master excel file with all 108 tables here: https://www.dropbox.com/scl/fi/qb303atvw6hzjdlmiiwf0/MLB_Complete_Dataset_2007-2026.xlsx?rlkey=k8shz22fwnllxf7bu6vz80mq6&st=r716c9wl&e=1&dl=0

Tableau Public PDS: https://public.tableau.com/app/profile/john.johansson/datasources


## At a glance

| | |
| --- | --- |
| Tables | 108 |
| Rows | 2,655,896 |
| Columns | 3,767, every one defined |
| Seasons | 20 (2007&ndash;2026) |
| Clubs | 30 |
| Size | 363 MB |
| Encoding | UTF-8 with BOM, so Excel opens them correctly |
| Format | CSV, comma delimited, `"` quote character |

| Category | Tables | Rows | What it covers |
| --- | ---: | ---: | --- |
| `regular_season` | 12 | 249,163 | Player and team data exactly as Baseball-Reference lays it out on each club's season page: batting, pitching, fielding, value, rosters, coaching staff, schedule and standings. |
| `playoffs` | 8 | 24,671 | The postseason equivalents, plus series results. Only clubs that qualified appear. |
| `advanced` | 52 | 382,827 | Advanced, situational, batted-ball and win-probability views, each at both player and team level. Sourced from league-wide pages, so they cover every player in the majors. |
| `standard_stats` | 15 | 128,965 | Ordinary season stat lines that Baseball-Reference publishes league-wide rather than per club, the team value/WAR totals, and the rookie, debut and free-agent cohorts. |
| `awards` | 5 | 1,941 | MVP, Cy Young, Rookie of the Year and Manager of the Year voting, plus the honours list. |
| `team_info` | 10 | 113,037 | Roster and front-office reference: transactions, free-agent signings, uniform numbers, opening-day and season lineups, debut biographies, and manager records and tendencies. |
| `projections` | 4 | 1,980 | Playoff odds and scenarios as published in-season, plus attendance and club miscellany. |
| `game_level` | 2 | 1,753,312 | One row per player per game: every batting order and every defensive alignment. |
| **Total** | **108** | **2,655,896** | |

## What is in the package

The 108 data tables sit in eight category folders, named above. Alongside them:

| File | What it is |
| --- | --- |
| `Data_Dictionary.csv` | Every one of the 3,767 columns in machine-readable form: category, table, column, the Baseball-Reference stat code it came from, and the site's own definition. |
| `FIELD_REFERENCE.md` | The same content as a readable document, organised table by table. |
| `datapackage.json` | A [Frictionless Data](https://specs.frictionlessdata.io/) tabular data package descriptor: typed schema for every column, row counts, byte sizes and a SHA-256 per file. |
| `data_join_diagrams` | Three SVG maps showing how the tables relate and which keys join them. |

### Two tables are split into parts

GitHub refuses files over 100 MB and warns above 50 MB, so the two game-level tables are
published in five parts each. Nothing else is split.

| Table | Parts | Rows | Part size |
| --- | ---: | ---: | --- |
| `Batting_Orders` | 5 | 847,098 | ~19 MB |
| `Defensive_Lineups` | 5 | 906,214 | ~20 MB |

Each part is a complete, valid CSV: it carries the BOM and the full header row, so it opens on
its own in Excel or any reader. The parts are a straight row-wise cut in the original order,
with no reordering and no dropped rows.

To work with a whole table, read the parts and concatenate:

```python
import pandas as pd, glob
batting_orders = pd.concat(
    (pd.read_csv(f, encoding="utf-8-sig") for f in sorted(glob.glob("game_level/Batting_Orders_part*.csv"))),
    ignore_index=True)
```

```r
library(data.table)
files <- sort(Sys.glob("game_level/Batting_Orders_part*.csv"))
batting_orders <- rbindlist(lapply(files, fread))
```

In Tableau or Power BI, point the connector at the folder and use a wildcard union on
`Batting_Orders_part*` &mdash; both tools will stack the parts and keep one header.

On the command line, skip the header of every part after the first:

```bash
head -1 Batting_Orders_part1.csv > Batting_Orders.csv
for f in Batting_Orders_part*.csv; do tail -n +2 "$f" >> Batting_Orders.csv; done
```

Reassembled this way, the result is byte-for-byte identical to the unsplit original.

### One merged data dictionary

The source carries a separate `Data_Dictionary.csv` inside each category folder. Those eight
files are merged here into a single `Data_Dictionary.csv` with a leading `Category` column, so
one lookup covers all 3,767 columns:

| Column | Meaning |
| --- | --- |
| `Category` | One of the eight folder names |
| `Dataset` | The table the column belongs to |
| `Column` | The column name as it appears in the CSV header |
| `Source_Stat_Code` | Baseball-Reference's own `data-stat` code for that column |
| `Description` | Baseball-Reference's definition |

## Conventions that apply to every file

**Identifier columns come first.** Most tables open with `Season`, `Team`, `Team_Name`,
`League`, `League_Name`, `Division`, `Division_Full`, and carry `Player_ID` and `Row_Type` at
the end. These were added during the build so each row stands alone &mdash; there is nothing to
look up.

**`Player_ID` is the Baseball-Reference player key** (`judgeaa01`). It is stable across seasons
and clubs, and it is the only safe way to join players. Names are not keys: the site decorates
them with handedness markers and roster-status suffixes, which were stripped into `Bats` and
`Throws` columns during the build.

**`Row_Type` separates data from footers.** `data` for real rows, `summary` for the totals rows
Baseball-Reference appends. 46 tables have them.

**`Rk` is a row number, not an identifier.** It means nothing outside its own file.

**Blank means not applicable.** It never means zero.

**Multi-club rows.** In the league-page categories a player who changed clubs mid-season gets a
row per club plus a combined row coded `2TM` or `3TM`. The `regular_season` category has no
combined rows, because it is scraped per club.

## How the tables join

There are no lookup tables and no surrogate keys. Every row carries its own identifiers, so any
two tables at the same grain join directly.

| Grain | Join on | What it means |
| --- | --- | --- |
| Team-season | `Season` + `Team` | 600 team-seasons: 30 clubs &times; 20 years |
| Player-team-season | `Season` + `Team` + `Player_ID` | a player's stint with one club in one year |
| Position | `Season` + `Team` + `Position` | add `Position` or rows multiply up to ten-fold |
| Game | `Season` + `Team` + `Gm#` or `Game_Num` | 94,122 team-games |
| Series | `Season` + `Series_Code` | postseason series; clubs via `Winning_Team` / `Losing_Team` |
| League-season | `Season` + `Lg` | the few tables published per league |

Three tables make natural starting points: **`Team_Season_Summary`** for anything team-level,
**`Full_Season_Roster`** for anything player-level, **`Schedule_Results`** for anything
game-level. `Player_ID` matches the roster 100% of the time from the core batting, pitching,
fielding and game-level tables.

Two rules stop most mistakes. **Join on `Player_ID`, never `Rk`.** And **never union a
`regular_season` table with an `advanced` one** &mdash; the first splits a traded player into one
row per club, the second adds a combined `2TM`/`3TM` row on top, so the player is counted twice.

### Join diagrams

`data_join_diagrams` holds three SVG maps of the same relationships, at increasing detail. They
open in any browser and are vector, so they stay sharp at any zoom.

| Diagram | Covers |
| --- | --- |
| `JOIN_DIAGRAM_General.svg` | One page. The handful of keys the whole dataset turns on, and how many of the 108 tables carry each: `Season` in all 108, `Team` in 107, `Player_ID` in 67, `Position` in 24, `Game_Num`/`Gm#` in 4. Start here. |
| `JOIN_MAP_20_Years_of_Modern_Baseball.svg` | The 54 core tables &mdash; everything except `advanced` &mdash; grouped by the grain they join at, with the exact key for each group. The practical map for most work. |
| `JOIN_MAP_All_Tables.svg` | All 108 tables across eight grains, with measured match rates between them. The complete reference. |

Every count and match rate on them was measured from the data rather than assumed.

## Caveats that change your numbers

Read these before you draw a conclusion. Each one has bitten a real analysis.

**Footer rows live in the same table as the data.** 46 of the 108 tables end with rows like
*Team Totals*, *Non-Pitcher Totals* or *Rank in 2 AL*. They are kept because they are useful,
but they sit alongside the players. `Row_Type` tells them apart: filter to `Row_Type = "data"`
before any sum, average or pivot, or you count each club twice.

**29 tables carry a League Average row** &mdash; 25 in `advanced`, 4 in `standard_stats` &mdash;
and it survives a `Row_Type = "data"` filter, because Baseball-Reference publishes it as a data
row rather than a footer. It has a blank `Team` and a `Team_Name` of *League Average*. It is why
the team-level tables in those categories hold 640 rows rather than 600. Exclude rows with no
`Team` before summing any counting stat, or you add a thirty-first club. Rates barely move;
totals do.

**19 tables carry a combined `TOT` row for a traded player as well as his per-club rows**
&mdash; 13 in `advanced`, 3 in `awards`, 3 in `standard_stats`. Summing across all rows
double-counts every player who moved mid-season. Keep the `TOT` row where one exists and drop
that player's club rows, or the reverse &mdash; never both. The `regular_season` and `playoffs`
categories have no `TOT` rows, so there summing club rows is correct.

**`Rk` is not an identifier.** It is the row number on the page it came from. Two tables can
share every `(Season, Team, Rk)` value and still describe different players at each one &mdash;
measured at only 3.9% agreement between `Standard_Batting` and `Value_Batting`. Join on
`Player_ID`.

**Innings pitched are written in thirds, not decimals.** `11.2` means eleven and two *thirds*,
not eleven and two tenths. The column sums without complaint and the total is quietly about
3.5% low, which inflates every ERA computed from it. Convert first:
`INT(IP) + ROUND((IP - INT(IP)) * 10) / 3`.

**2020 was a 60-game season.** COVID-shortened, so counting stats are roughly 37% of a normal
year, and no player reaches the usual 502 plate-appearance or 162-inning qualifying thresholds.
Compare rates, not totals, and scale any qualifier you apply. Attendance is blank because games
were played without spectators.

**2026 is the regular season only.** The postseason had not started when this was built, so
every `Postseason_*` table stops at 2025. Award voting is not announced until November, so the
five awards tables stop at 2025 too. Within the 2026 regular season, every club has 162 games
recorded except Baltimore and the Yankees, which have 161 &mdash; one game between them was still
outstanding.

**Four franchise identities shift inside this window.** The Athletics are `OAK` through 2024 and
`ATH` from 2025; the Marlins are `FLA` to 2011 and `MIA` from 2012; the Rays are `TBD` in 2007
and `TBR` from 2008. Houston moved from the NL Central to the AL West in 2013, so league-level
totals must expect it to change sides. Group on the code pairs to trend a club across all 20
years, or a renamed club splits into two short records.

**The designated hitter was not universal until 2022**, so before then &mdash; 2020 excepted &mdash;
National League pitchers batted.

**The postseason field kept changing size**: 8 clubs through 2011, 10 from 2012, 16 in the
expanded 2020 season, 10 in 2021, 12 from 2022. Postseason counting stats are not comparable
across those formats, and a larger field admits weaker clubs, which moves league-wide postseason
rates for structural rather than competitive reasons.

**Postseason samples are small.** A postseason holds a few thousand plate appearances against
roughly 185,000 in the regular season &mdash; about 60 to 1. A twenty-point swing in a postseason
rate is well inside noise.

**A blank cell means "not applicable", not zero.** Four columns carry values for 2026 only
&mdash; `Strk`, `last10`, `last20`, `last30` on the expanded standings, and `Preview` and
`Starttime` on the schedule. Baseball-Reference renders those only while a season is current, so
they are point-in-time readings, not season summaries.

**Eleven tables have an empty `Team` column on every row.** They come from league-wide pages,
so the club sits in the source's own `Tm` column instead, or you reach the rows through
`Player_ID`. They are `Awards_And_Honors`, `Cumulative_Batting`, `Cumulative_Pitching`,
`Free_Agent_Batting`, `Free_Agent_Pitching`, `Free_Agent_Signings`, `Neutral_Batting`,
`Neutral_Pitching`, `Opening_Day_Lineups`, `Playoff_Odds` and `Playoff_Scenarios`.

**`Current_40_Man_Roster` is a live snapshot**, marked Season 2026, not a historical series.

**Statcast metrics are not included.** Baseball-Reference does not publish them, so no exit
velocity, launch angle or expected statistics appear anywhere in this dataset.

## The categories in detail

Grain is the smallest unique column set, measured over the data rows. "event log" means the
table has no single key &mdash; it is a list of occurrences rather than one row per entity.

### `regular_season` &mdash; 12 tables, 249,163 rows

Player and team data exactly as Baseball-Reference lays it out on each club's season page.

| Table | Rows | Columns | Grain |
| --- | ---: | ---: | --- |
| `Coaching_Staff` | 5,969 | 16 | event log |
| `Current_40_Man_Roster` | 1,358 | 23 | `Season + Team + Rk` |
| `Full_Season_Roster` | 30,100 | 37 | `Season + Team + Rk` |
| `Schedule_Results` | 94,188 | 34 | `Season + Team + Gm#` |
| `Standard_Batting` | 24,760 | 43 | `Season + Team + Rk` |
| `Standard_Fielding` | 31,522 | 38 | `Season + Team + Rk + Player` |
| `Standard_Pitching` | 17,353 | 46 | `Season + Team + Rk` |
| `Standings` | 600 | 13 | `Season + Team` |
| `Standings_Expanded` | 600 | 36 | `Season + Team` |
| `Team_Season_Summary` | 600 | 27 | `Season + Team` |
| `Value_Batting` | 24,760 | 31 | `Season + Team + Rk` |
| `Value_Pitching` | 17,353 | 33 | `Season + Team + Rk` |

### `playoffs` &mdash; 8 tables, 24,671 rows

The postseason equivalents, plus series results. Only clubs that qualified appear.

| Table | Rows | Columns | Grain |
| --- | ---: | ---: | --- |
| `Postseason_Advanced_Batting` | 4,670 | 27 | `Season + Team + Rk` |
| `Postseason_Advanced_Pitching` | 2,120 | 31 | `Season + Team + Rk` |
| `Postseason_Batting` | 4,845 | 38 | `Season + Team + Rk` |
| `Postseason_Fielding` | 4,501 | 30 | `Season + Team + Rk` |
| `Postseason_Pitching` | 2,295 | 43 | `Season + Team + Rk` |
| `Postseason_Roster` | 4,651 | 37 | `Season + Team + Rk` |
| `Postseason_Schedule_Results` | 1,414 | 27 | `Season + Team + Gm#` |
| `Postseason_Series_Results` | 175 | 13 | `Season + Series_Code` |

`Postseason_Series_Results` is one row per series with `Winning_Team` and `Losing_Team` as
separate columns, so a club appears in whichever column applies. Pivot those two columns to get
one row per club per series.

### `advanced` &mdash; 52 tables, 382,827 rows

Advanced, situational, batted-ball and win-probability views, each at both player and team
level. Sourced from league-wide pages, so they cover every player in the majors.

| Table | Rows | Columns | Grain |
| --- | ---: | ---: | --- |
| `Advanced_Batting` | 25,104 | 39 | event log |
| `Advanced_Fielding_1B` | 3,772 | 47 | `Season + Team + Player_ID` |
| `Advanced_Fielding_2B` | 3,537 | 46 | `Season + Team + Player_ID` |
| `Advanced_Fielding_3B` | 3,654 | 44 | `Season + Team + Player_ID` |
| `Advanced_Fielding_C` | 2,383 | 42 | `Season + Team + Player_ID` |
| `Advanced_Fielding_CF` | 3,369 | 52 | `Season + Team + Player_ID` |
| `Advanced_Fielding_C_Baserunning` | 2,383 | 33 | `Season + Team + Player_ID` |
| `Advanced_Fielding_LF` | 5,039 | 52 | `Season + Team + Player_ID` |
| `Advanced_Fielding_P` | 16,773 | 37 | `Season + Team + Rk` |
| `Advanced_Fielding_RF` | 4,526 | 52 | `Season + Team + Player_ID` |
| `Advanced_Fielding_SS` | 2,686 | 47 | `Season + Team + Player_ID` |
| `Advanced_Pitching` | 18,351 | 33 | event log |
| `Baserunning_Batting` | 20,572 | 46 | `Season + Team + Rk` |
| `Basesituation_Pitching` | 18,390 | 47 | `Season + Team + Rk` |
| `Batting_Pitching` | 18,452 | 40 | `Season + Team + Rk` |
| `Neutral_Batting` | 17,453 | 32 | `Season + Team + Rk` |
| `Neutral_Pitching` | 15,131 | 33 | `Season + Team + Rk` |
| `Pitches_Batting` | 20,514 | 43 | `Season + Team + Rk` |
| `Pitches_Pitching` | 18,390 | 44 | `Season + Team + Rk` |
| `Ratio_Batting` | 20,514 | 30 | `Season + Team + Rk` |
| `Ratio_Pitching` | 18,452 | 32 | `Season + Team + Rk` |
| `Reliever_Pitching` | 14,869 | 45 | `Season + Team + Rk` |
| `Sabermetric_Batting` | 25,559 | 37 | `Season + Team + Rk` |
| `Situational_Batting` | 20,514 | 52 | `Season + Team + Rk` |
| `Starter_Pitching` | 7,488 | 47 | `Season + Team + Rk` |
| `Win_Probability_Batting` | 20,500 | 35 | `Season + Team + Rk` |
| `Win_Probability_Pitching` | 18,452 | 36 | `Season + Team + Rk` |
| `Team_Advanced_Batting` | 640 | 31 | `Season + Team` |
| `Team_Advanced_Fielding_1B` | 640 | 43 | `Season + Team` |
| `Team_Advanced_Fielding_2B` | 640 | 42 | `Season + Team` |
| `Team_Advanced_Fielding_3B` | 640 | 40 | `Season + Team` |
| `Team_Advanced_Fielding_C` | 640 | 38 | `Season + Team` |
| `Team_Advanced_Fielding_CF` | 640 | 48 | `Season + Team` |
| `Team_Advanced_Fielding_C_Baserunning` | 640 | 29 | `Season + Team` |
| `Team_Advanced_Fielding_LF` | 640 | 48 | `Season + Team` |
| `Team_Advanced_Fielding_P` | 640 | 33 | `Season + Team` |
| `Team_Advanced_Fielding_RF` | 640 | 48 | `Season + Team` |
| `Team_Advanced_Fielding_SS` | 640 | 43 | `Season + Team` |
| `Team_Advanced_Pitching` | 640 | 26 | `Season + Team` |
| `Team_Baserunning_Batting` | 640 | 41 | `Season + Team` |
| `Team_Basesituation_Pitching` | 640 | 42 | `Season + Team` |
| `Team_Batting_Pitching` | 640 | 35 | `Season + Team` |
| `Team_Pitches_Batting` | 640 | 39 | `Season + Team` |
| `Team_Pitches_Pitching` | 640 | 39 | `Season + Team` |
| `Team_Ratio_Batting` | 640 | 25 | `Season + Team` |
| `Team_Ratio_Pitching` | 640 | 27 | `Season + Team` |
| `Team_Reliever_Pitching` | 640 | 39 | `Season + Team` |
| `Team_Sabermetric_Batting` | 640 | 31 | `Season + Team` |
| `Team_Situational_Batting` | 640 | 47 | `Season + Team` |
| `Team_Starter_Pitching` | 640 | 41 | `Season + Team` |
| `Team_Win_Probability_Batting` | 640 | 30 | `Season + Team` |
| `Team_Win_Probability_Pitching` | 640 | 31 | `Season + Team` |

Thirteen tables in this category carry a combined `TOT` row for a traded player **as well as**
his per-club rows, and 25 carry a League Average row with a blank `Team`. Both survive a
`Row_Type = "data"` filter. See the caveats above before you sum anything here.

### `standard_stats` &mdash; 15 tables, 128,965 rows

Ordinary season stat lines that Baseball-Reference publishes league-wide rather than per club,
the team value/WAR totals, and the rookie, debut and free-agent cohorts.

| Table | Rows | Columns | Grain |
| --- | ---: | ---: | --- |
| `Cumulative_Batting` | 29,590 | 38 | `Season + Team + Rk` |
| `Cumulative_Pitching` | 16,856 | 43 | `Season + Team + Rk` |
| `Debuts_Batting` | 4,861 | 37 | `Season + Team + Player_ID` |
| `Debuts_Pitching` | 3,152 | 42 | `Season + Team + Player_ID` |
| `Free_Agent_Batting` | 639 | 32 | `Season + Team + Player_ID` |
| `Free_Agent_Pitching` | 979 | 36 | `Season + Team + Player_ID` |
| `Rookies_Batting` | 4,251 | 37 | `Season + Team + Player_ID` |
| `Rookies_Pitching` | 2,784 | 42 | `Season + Team + Player_ID` |
| `Standard_Fielding_By_Position` | 56,293 | 53 | event log |
| `Team_Fielding_By_Position` | 6,400 | 48 | `Season + Team + Position` |
| `Team_Standard_Batting` | 640 | 37 | `Season + Team` |
| `Team_Standard_Fielding` | 640 | 27 | `Season + Team` |
| `Team_Standard_Pitching` | 640 | 44 | `Season + Team` |
| `Team_Value_Batting` | 620 | 27 | `Season + Team` |
| `Team_Value_Pitching` | 620 | 29 | `Season + Team` |

Four tables here carry a League Average row with a blank `Team`, which is why the team-level
tables hold 640 rows rather than 600: 30 clubs plus that row in each of the 20 seasons.
`Rookies_Batting`, `Rookies_Pitching` and `Standard_Fielding_By_Position` also carry `TOT` rows
for traded players.

### `awards` &mdash; 5 tables, 1,941 rows

MVP, Cy Young, Rookie of the Year and Manager of the Year voting, plus the honours list.
Voting is announced in November, so these stop at 2025.

| Table | Rows | Columns | Grain |
| --- | ---: | ---: | --- |
| `Awards_And_Honors` | 102 | 12 | `Season + Team + Award` |
| `Cy_Young_Voting` | 375 | 40 | `Season + Team + Player_ID` |
| `MVP_Voting` | 911 | 40 | `Season + Team + Player_ID` |
| `Manager_Of_The_Year_Voting` | 246 | 21 | `Season + Team` |
| `Rookie_Of_The_Year_Voting` | 307 | 40 | `Season + Team + Player_ID` |

### `team_info` &mdash; 10 tables, 113,037 rows

Roster and front-office reference: transactions, free-agent signings, uniform numbers,
opening-day and season lineups, debut biographies, and manager records and tendencies.

| Table | Rows | Columns | Grain |
| --- | ---: | ---: | --- |
| `Debuts_Bio` | 4,841 | 27 | `Season + Team + Player_ID` |
| `Free_Agent_Signings` | 8,038 | 41 | `Season + Team + Rk` |
| `Manager_Record` | 680 | 25 | `Season + Team + Player_ID` |
| `Manager_Tendencies` | 680 | 36 | `Season + Team + Player_ID` |
| `Opening_Day_Lineups` | 600 | 21 | `Season + Team + Player_ID` |
| `Team_Appearances` | 600 | 27 | `Season + Team` |
| `Team_Pitching_Staffs` | 600 | 20 | `Season + Team` |
| `Team_Starting_Lineups` | 600 | 20 | `Season + Team` |
| `Transactions` | 66,173 | 8 | event log |
| `Uniform_Numbers` | 30,225 | 9 | `Season + Team + Uniform_Number + Player` |

### `projections` &mdash; 4 tables, 1,980 rows

Playoff odds and scenarios as published in-season, plus attendance and club miscellany.
The odds and scenarios are point-in-time readings from when the pages were fetched, not
end-of-season outcomes.

| Table | Rows | Columns | Grain |
| --- | ---: | ---: | --- |
| `Playoff_Odds` | 720 | 33 | event log |
| `Playoff_Scenarios` | 40 | 17 | `Season + Team + League` |
| `Team_Attendance` | 620 | 15 | `Season + Team` |
| `Team_Miscellaneous` | 600 | 26 | `Season + Team` |

`Team_Miscellaneous` carries average game time, estimated payroll, batting and pitching age,
park factors and manager challenge records. Its `Succ%` column is **manager replay-challenge
success**, not stolen base success &mdash; a column name that has caught people out.

### `game_level` &mdash; 2 tables, 1,753,312 rows

One row per player per game: every batting order and every defensive alignment. Published in
five parts each, as described above.

| Table | Rows | Columns | Grain |
| --- | ---: | ---: | --- |
| `Batting_Orders` | 847,098 | 18 | `Season + Team + Game_Num + Player_ID` |
| `Defensive_Lineups` | 906,214 | 17 | `Season + Team + Game_Num + Position` |

## How the data was collected

Every figure is scraped from Baseball-Reference.com. Pages were fetched over HTTP one at a time
at a 3-second crawl delay, honouring the `Crawl-delay: 3` directive in the site's `robots.txt`.
Only `/teams/`, `/leagues/` and `/awards/` paths were requested. Player split pages, which
`robots.txt` disallows, were never touched.

Tables embedded inside HTML comments &mdash; Baseball-Reference hides many of its secondary
tables that way &mdash; were extracted along with the visible ones. Every column is keyed back to
the site's own `data-stat` code, preserved in the dictionary as `Source_Stat_Code`, so any
column can be traced to the page it came from.

Nothing is derived, imputed or recalculated. Values are as the source published them.

## A note on the package descriptor

`datapackage.json` describes the dataset as it was built, with the two game-level tables as
single files. Their entries list the unsplit byte size, row count and SHA-256, which will not
match the part files on disk. Every other entry matches exactly. If you reassemble the parts as
shown above, the hashes for those two tables match as well.

## Source and terms of use

Data from **[Baseball-Reference.com](https://www.baseball-reference.com/)**

 

Suggested citation:

> Baseball-Reference.com, Sports Reference LLC. MLB season data, 2007&ndash;2026.
> Retrieved 2026-10-01.
