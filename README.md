# F1 Power BI Dashboard: From Race History to the 2026 Grid

An interactive six-page Power BI dashboard built on 15 seasons of Formula 1 data (2011–2025, 310 races). It starts with season-level trends and lets you drill down to individual drivers, constructors and circuits, including profile pages for the 2026 grid.

· Chaitanya Kewadkar · [LinkedIn](https://www.linkedin.com/in/chaitanyakew/) ·

![Landing page](Dashboard%20Images/1.%20Landing%20page.png)

## What it does

- **Races overview:** champions, race wins and points for any season or for all 15 seasons.
- **Sprint overview:** sprint wins by driver and constructor, 2021–2025.
- **Constructor profiles:** one page for each of the 11 teams on the 2026 grid, with logo, car, career stats and last win.
- **Driver profiles:** one page for each 2026 driver, with portrait, helmet, team, career stats and last win.
- **Circuit profiles:** previous winners, fastest lap and location for every 2026 circuit.

## Tools

- Power BI Desktop
- Power Query (M) for cleaning and loading
- DAX for measures
- Star-schema data model

## Pages

### 1. Landing page
A navigation page with five buttons: Races overview, Sprint overview, Drivers for 2026, Constructors for 2026 and Circuits for 2026. Every other page has a back arrow in the top-right corner.

### 2. Races overview
Year buttons act as a multi-select slicer (**Select all** or any single season). Everything on the page responds to it:

- Total races
- Formula 1 Champion and Constructors Champion, each with total points
- Driver and constructor with the most wins
- Grand Prix wins by driver and by constructor
- Top 5 drivers and top 3 constructors by points

The two views below show all seasons and 2023.

| All 15 seasons | 2023 only |
|---|---|
| ![15 year overview](Dashboard%20Images/2.%2015%20year%20Overview.png) | ![Season 2023](Dashboard%20Images/3.%20Season%202023.png) |

### 3. Sprint overview
Sprint races have run since 2021, so the slicer covers 2021–2025. It shows the number of sprints held and total sprint wins by driver and by constructor.

| 2021–2025 | 2025 only |
|---|---|
| ![Sprint overview](Dashboard%20Images/4.%20Sprint%20Overview.png) | ![Sprint 2025](Dashboard%20Images/5.%20Sprint%202025.png) |

### 4. Constructors for 2026
Pick a team from the tile strip along the top. The whole page re-themes to that team's colour. For the selected team it shows:

- Logo and car image
- Total Grands Prix, Grand Prix wins, Q2 appearances, Q3 appearances, Q1–Q3 conversion and total laps
- The last race won, with the date and the Grand Prix name
- Laps driven by each driver for the team
- A table of the team's winning drivers and their wins

| Ferrari | Red Bull |
|---|---|
| ![Constructor Ferrari](Dashboard%20Images/6.%20Constructor%201.png) | ![Constructor Red Bull](Dashboard%20Images/7.%20Constructor%202.png) |

### 5. Drivers for 2026
Pick a driver from the list on the right. The page shows the portrait, race number, three-letter code, team logo, helmet and team colour, plus:

- Nationality and age
- Total Grands Prix, podiums, Grand Prix wins, sprint wins and laps driven
- The last race won, with the date
- Q2 and Q3 appearances and the Q1–Q3 conversion rate

| Lewis Hamilton | Max Verstappen |
|---|---|
| ![Driver Hamilton](Dashboard%20Images/8.%20Driver%201.png) | ![Driver Verstappen](Dashboard%20Images/9.%20Driver%202.png) |

### 6. Circuits for 2026
Pick a circuit from the list. The page shows the country and circuit, how many Grands Prix were held there, a table of previous winners (year, driver, constructor), the years the race was held, the fastest lap (time, driver and date) and a map of the location.

![Circuits](Dashboard%20Images/10.%20Circuits.png)

## Data model

A star schema with fact tables (Results, SprintResults, Qualifying, ConstructorResults, LapTimes) around the dimensions (Drivers, Constructors, Circuits, Races, Seasons). Relationships are many-to-one and single-direction. A small set of side tables (Drivers2026, Constructors26, Circuits26 and the image tables) drives the 2026 profile pages (I created these for the dashboard design; they are not part of the original dataset).

![Data model](Dashboard%20Images/11.%20Data%20Model.png)

## Data preparation (Power Query)

- A `FolderPath` parameter and a reusable `fnLoadCsv` function load every CSV the same way: promote headers, then turn the `\N` null marker into real nulls.
- `Races` is filtered to 2011–2025 (310 rows). Fact tables are scoped to the same years by an inner join to `Races`, so the whole model stays consistent.
- Display columns such as `DriverName` and `ConstructorName` are created in Power Query.
- The 2026 lookup tables (`Drivers2026`, `Constructors26`, `Circuits26`) are built from the raw 2026 entries.
- Driver_images and constructor_images tables hold the portrait, helmet, logo, car and team-colour data.

```powerquery
(fileName as text) as table =>
let
    Source   = Csv.Document(File.Contents(FolderPath & fileName),
                 [Delimiter = ",", Encoding = 65001, QuoteStyle = QuoteStyle.Csv]),
    Promoted = Table.PromoteHeaders(Source, [PromoteAllScalars = true]),
    NoNulls  = Table.ReplaceValue(Promoted, "\N", null, Replacer.ReplaceValue,
                 Table.ColumnNames(Promoted))
in
    NoNulls
```

## DAX measures

Measures are grouped into display folders:

| Folder | What the measures do |
|---|---|
| **Races - Top** | Champions and records: top driver and constructor by points and by wins, with the points or win count |
| **Driver26** | Driver profile card: age, number, team, podiums, Grand Prix and sprint wins, laps driven, last win date and race, Q1/Q2/Q3 appearances, Q3 rate, and image and team-colour measures |
| **Constructor 26** | Team profile card: total Grands Prix, wins, total laps, last win date and race, logo and car image URLs, and team colours |
| **Circuit26** | Circuit profile: total Grands Prix held and the fastest lap (time, driver, date) |

Techniques used:

- **Last-win tracking:** finds the latest race a driver or team won by taking the latest race date among their winning results.
- **Qualifying conversion:** Q3 rate is Q3 appearances divided by Q1 appearances, so it shows how often a driver or team makes the final qualifying session.
- **Dynamic theming:** colour measures return a team's hex colour for use in conditional formatting, so the page re-themes to the selected team or driver.
- **Dynamic images:** image measures return URLs, with the data category set to Image URL, to show logos, cars, portraits and helmets for the current selection.

## How to open

1. Install Power BI Desktop (free).
2. Open `F1.pbix`.
3. To refresh, go to **Transform data → Edit parameters** and set `FolderPath` to the folder holding the CSV files (Dataset Fodler in Repository), with a trailing backslash.
4. In **File → Options → Current File → Privacy**, "Ignore the Privacy Levels" must be on, or the fact-table queries fail with a firewall error.

## Notes and limitations

- The model covers 2011–2025. The 2026 season is only used to choose which drivers, teams and circuits get profile pages, so 2026 rookies show blank values and 2026 results are not counted.
- "All-time" figures mean 2011–2025 only.

## Data credits

- Race data: [Formula 1 Race Data](https://www.kaggle.com/datasets/jtrotman/formula-1-race-data) by jtrotman on Kaggle.
- Driver portraits, helmets and numbers: turn1visuals.com.
- Team logos, car images and Formula 1 imagery belong to their owners. They are used here for a personal portfolio project. This is not an official Formula 1 product.
