# What made a Roman city big?

**Was it geography, or Rome itself?** In this project, I study 1,362 cities of the Roman Empire around AD 200 to find out which signals are linked to a city's size, and I use a model to estimate the size of the cities nobody has measured yet.

Lucas Vieira · [github.com/lucasnvieira](https://github.com/lucasnvieira)

- Notebook: [`notebooks/roman_cities.ipynb`](notebooks/roman_cities.ipynb)
- Blog post: [lucasnvieira.github.io/roman-cities](https://lucasnvieira.github.io/roman-cities/)

Project for the Udacity Data Scientist Nanodegree ("Write a Data Science Blog Post").

## Motivation

Roman cities varied a lot in size: Rome covered about 1,783 hectares, while the typical town had about 31. Archaeologists have measured the area of only 865 of the 1,362 cities active in AD 200. I wanted to know whether a city's size can be explained by where it was (coast, rivers, roads, mines, distance from Rome) or by what Rome made of it (civic status and public buildings). I chose this topic because history is a field where data storytelling can reach people who would never open a spreadsheet.

I picked AD 200 as a single snapshot because it is the peak of the empire, just before the Crisis of the Third Century (AD 235 to 284), when more than twenty emperors, invasions, plague and the loss of value of the silver coin changed the Roman world.

## Questions and findings

All the numbers below come from the notebook. Model results are from a cross-validation **by province** (all the cities of a province are tested together), which is the honest score for a region the model has never seen.

| # | Question | Finding |
| --- | --- | --- |
| 1 | Where and when did the Roman world become urban? | Only 180 of the 1,362 cities (13%) were founded in the imperial era. The Greek East was urban centuries earlier (median foundation 323 BC); Rome urbanised the West mostly in the 1st century BC (366 western cities). |
| 2 | Was it geography or Rome itself that made a city big? | Rome. The strongest signals are the variety of recorded monuments and being a capital (median 63 ha against 25.5 ha). Geography alone explains almost nothing (R² 0.03). Surprise: cities far from Rome tend to be bigger (median 41 ha in the farthest quarter by travel time, 26 ha in the closest). |
| 3 | How well can we guess a city's size? | R² 0.23 on the log of the area, median error 16.2 ha, and 60.0% of the cities predicted within a factor of 2. Good for orders of magnitude, not for exact areas. |
| 4 | How big were the cities nobody has measured? | The 497 unmeasured cities look like smaller towns (median estimate 19.3 ha, against 29.7 ha for the measured cities through the same model). Exception: 8 of the 10 largest estimates are district capitals in Egypt, at about 160 ha. |
| 5 | Theatres in the East, amphitheatres in the West? | Yes. A theatre is recorded in 65.9% of eastern cities against 36.1% in the West; an amphitheatre in 33.1% of western cities against 6.9% in the East. |

## Method (CRISP-DM)

The notebook is organised by the steps of CRISP-DM:

1. **Business understanding:** the five questions, the AD 200 snapshot and the decisions about the data.
2. **Data understanding:** gather and assess each dataset (missing values, duplicates, dates, the skewed target).
3. **Data preparation:** keep the cities active in AD 200; turn civic status and free-text monuments into binary features; connect every dataset **by location** (distance to the nearest road, coast and river; mines within 50 km; travel days and cost to Rome on the ORBIS network).
4. **Modelling:** linear regression and random forest (scikit-learn) on the log of the area, with two feature sets: *place and time* only, and *everything* (adding status and monuments). Permutation importance shows which features matter.
5. **Evaluation:** random folds against folds by province, plus robustness checks (mine radius of 25 and 100 km, model without roads, civic status without uncertain titles). None of them changes the conclusions.
6. **Communication:** the [blog post](https://lucasnvieira.github.io/roman-cities/), written for a non-technical audience.

### Notable data decisions

- **Target in log scale.** The area goes from 1 to 1,783 ha with a median of 31 ha, so I model the logarithm to keep a few giant cities from dominating the model.
- **Missing target is not imputed.** The 865 measured cities train and test the model; the 497 unmeasured cities are the ones predicted in question 4.
- **Missing features.** After the spatial enrichment there are no missing values left; the pipeline still has a median imputer as a safety net, because medians are not pulled by extreme distances.
- **Categorical variables.** Civic titles become four 0/1 flags (colonia, municipium, capital, other rights) plus a flag for "no status recorded"; monuments become one 0/1 flag per building type. Titles marked with `?` in the source count as probable.
- **Leakage.** I excluded the Barrington Atlas rank (it partly reflects city size) and the presence of walls (361 areas were measured from the line of the walls).
- **ORBIS network rebuilt.** Linking the ORBIS routes by their site ids leaves the network split into 48 pieces, because more than 500 ids are sea and river waypoints missing from the sites file. I rebuilt the network from the route geometry (route ends closer than about 5 km are merged). The fastest trips from Rome then come out at about 27 days to London and 15 days to Alexandria.
- **Coherence between periods.** ORBIS and the cities are dated to the same period. The roads have no date per segment, and only 49 of the 551 mines have a date; removing the roads makes the model slightly better (R² 0.25), so the conclusions do not depend on the undated layers.
- **No coin hoards.** I planned to use the *Coin Hoards of the Roman Empire* database as a sign of wealth, but its website was offline. I decided not to look for an alternative source, because people buried their savings in times of danger, so surviving hoards record insecurity (and the third-century crisis itself) more than prosperity.

## Files

```
.
├── README.md
├── LICENSE                                    MIT license for the code
├── requirements.txt                           Python libraries and versions
├── notebooks/
│   └── roman_cities.ipynb                     analysis, models and answers (CRISP-DM)
├── data/
│   ├── raw/
│   │   ├── hanson2016_cities.csv              city catalogue: names, province, dates, coordinates
│   │   ├── hanson2016_areas.csv               measured area (ha) and basis of the estimate
│   │   ├── hanson2016_civic_status.csv        civic titles (several rows per city)
│   │   ├── hanson2016_monuments.csv           recorded buildings (free text)
│   │   ├── hanson2016_bibliography.csv        references of the catalogue
│   │   ├── orbis_sites_extended.csv           ORBIS sites (including Rome)
│   │   ├── orbis_base_routes.geojson          ORBIS routes with travel days and cost
│   │   ├── oxrep_mines.csv                    551 Roman mines with coordinates and metals
│   │   ├── awmc_cultural_shapefiles_apr_2024.zip   AWMC roads and other cultural layers
│   │   └── awmc_physical_shapefiles_apr_2024.zip   AWMC coastline, rivers and other physical layers
│   └── processed/
│       ├── roman_cities_ad200_features.csv    one row per city active in AD 200, with all features
│       └── unmeasured_cities_predictions.csv  estimated area and 80% range for the unmeasured cities
├── figures/                                   the cover image and the five charts (made by the notebook)
├── site/
│   └── index.html                             the blog post, published with GitHub Pages
└── .github/workflows/pages.yml                copies the charts into the site and deploys it to GitHub Pages
```

## Libraries

Python 3.13 with pandas, NumPy, SciPy, GeoPandas (with pyogrio and Shapely), NetworkX, scikit-learn and Matplotlib. Exact versions are in [`requirements.txt`](requirements.txt).

## Reproduce it on your computer

```bash
git clone <this repository>
cd <repository folder>
python -m venv .venv
source .venv/bin/activate        # on Windows: .venv\Scripts\activate
pip install -r requirements.txt
cd notebooks
jupyter notebook roman_cities.ipynb
```

Run the notebook from the `notebooks` folder: it reads the data with relative paths (`../data/raw`) and writes the outputs to `../data/processed` and `../figures`. A full run takes about two minutes.

## Limitations

- **Research bias:** regions with more excavation have more recorded monuments and more measured areas.
- **No record is not absence:** a missing monument may simply be unexcavated.
- **Signs, not causes:** monuments and titles show a big city; they do not prove what made it big.
- **Small sample:** 865 measured cities, so individual estimates have wide ranges (80% of errors fall between 0.31x and 3.13x).

## Acknowledgements and data sources

- **Cities:** J. W. Hanson, *An Urban Geography of the Roman World, 100 B.C. to A.D. 300* (Archaeopress, 2016), database published by the [Oxford Roman Economy Project (OXREP)](https://oxrep.web.ox.ac.uk/cities-database).
- **Transport network:** W. Scheidel and E. Meeks, [ORBIS: The Stanford Geospatial Network Model of the Roman World](https://github.com/emeeks/orbis_v2).
- **Roads, coastline and rivers:** [Ancient World Mapping Center (AWMC), University of North Carolina](https://github.com/AWMC/geodata), derived from the *Barrington Atlas of the Greek and Roman World*; Open Database License (ODbL).
- **Mines:** [OXREP Mines Database](http://oxrep.classics.ox.ac.uk/databases/mines_database/), coordinated by Andrew Wilson.
- **Dataset list:** [Project Mercury](https://projectmercury.eu/datasets/), suggested by Udacity for this project.

The code is mine and is shared under the [MIT license](LICENSE); the data belongs to its original sources and keeps their licenses.

## AI assistance disclosure

I used Claude Code (Anthropic) as an assistant in this project. It helped me write and debug the Python code, collect the datasets, and draft the text of the notebook, this README and the blog post. I chose the topic and the data sources, defined the questions, gave the instructions for each step and reviewed the results and the final text. Since English is not my native language, the AI also helped me review the grammar and keep the text clear and consistent.
