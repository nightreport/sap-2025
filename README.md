# World Happiness Report: What Predicts National Happiness?

Applied statistics coursework in R. Hypothesis testing and multiple linear regression on
World Happiness Report data for 137 countries, comparing the 2023 and 2025 editions and
modelling happiness against socioeconomic indicators.

**Authors:** Tamara Ćorić, Ana Marić, Gregor Mihaljević, Petar Kuruc
**Course:** Statistička analiza podataka (Statistical Data Analysis), University of Zagreb, FER
**Date:** January 2026
**Language:** the report is written in Croatian; this README summarises it in English.

---

## Data

The Cantril ladder score from the World Happiness Report, joined to socioeconomic
indicators from several sources: GDP per capita and Gini coefficient from the World Bank,
healthy life expectancy and alcohol consumption from the WHO, a national healthcare score
from the Legatum Prosperity Index, and a crime index. Social support, freedom to make life
choices, generosity and perceptions of corruption come from the Gallup World Poll. Full
descriptions are in `Opis Varijabli.csv`.

The 2023 and 2025 tables were joined manually, because the same countries appear under
differing names across the two editions. Ten countries present only in the 2025 data
(Azerbaijan, Belize, Eswatini, Kuwait, Lesotho, Libya, Oman, Somalia, Trinidad and Tobago,
Yemen) were removed, leaving 137 countries observed in both years.

Some countries are missing values for individual indicators. Where the analysis required
complete cases, the handling is stated below.

---

## Questions and results

### 1. Did national happiness change between the 2023 and 2025 reports?

The same countries observed at two time points, so a paired t-test. Normality of the paired
differences was assessed from a histogram and a Q-Q plot before testing.

```
t = 3.332, df = 136, p-value = 0.001111
```

**Result:** the difference is statistically significant, so we reject the null. It is also
tiny, at roughly 0.07 points on a scale from 0 to 10. The two are worth separating: a
significant result across 137 paired observations says the shift is real, not that it means
anything at the level of an individual.

### 2. Can happiness be predicted from the other variables, and which predicts it best?

Missing values were replaced with the column median rather than dropped, since dropping
rows would have shrunk an already small dataset and risked bias. Predictors were screened
for multicollinearity with a correlation matrix before fitting.

| Model | Adjusted R² | F | p |
|---|---|---|---|
| Full, 9 predictors | 0.8355 | 77.74 on 9 and 127 DF | < 2.2e-16 |
| Reduced and standardised, 8 predictors | 0.8192 | 78.01 on 8 and 128 DF | < 2.2e-16 |

The full model fits slightly better, but the healthcare and crime indices are badly
collinear with the rest, which makes individual coefficients unsafe to interpret. We
dropped the crime index, standardised the remaining variables so their coefficients are
comparable to one another, and treat the second model as the one to read.

Model quality was checked with a predicted-versus-actual plot and a residuals-versus-fitted
plot with a loess smoother.

**Result:** roughly 82% of the variation in national happiness is explained. On standardised
coefficients the strongest positive predictors are social support, freedom to make life
choices, and GDP per capita; the strongest negative one is perceived corruption.

### 3. Does happiness differ between world regions?

The report's ten regions were collapsed into five (Americas and ANZ, Asia, Europe, Middle
East / North Africa / CIS, Sub-Saharan Africa), because several of the original regions held
too few countries to test. 2025 scores only.

Shapiro-Wilk on the ANOVA residuals returns p = 8.01e-05, so the normality assumption is
formally rejected. We proceeded with one-way ANOVA regardless: across 137 observations in
five groups of reasonable size ANOVA is robust to departures of this kind, and the effect
below is far too large to be an artefact of non-normal residuals.

```
F = 39.61 on 4 and 132 DF, p < 2e-16
```

**Result:** happiness differs significantly between regional groups.

### 4. Is the direction of change independent of region?

For each country we took the 2025 score minus the 2023 score and labelled it an improvement
or a deterioration, then cross-tabulated that against regional group. Before testing we
confirmed every expected cell frequency was at least 5, which is a condition `chisq.test()`
assumes rather than checks.

```
X-squared = 9.909, df = 4, p-value = 0.04199
```

**Result:** we reject independence at the 5% level, so which direction a country moved is
associated with its region. This one sits just under the threshold, making it suggestive
rather than firm.

---

## Repository contents

| File | Description |
|---|---|
| `WHR_sap.Rmd` | R Markdown source: all code, tests and commentary |
| `WHR_sap.pdf` | Knitted report with full output and plots |
| `WHR_2023_2025.csv` | Merged dataset, 137 countries, used by most of the analysis |
| `WHR_2023.csv` | 2023 report with socioeconomic indicators |
| `WHR_2025.csv` | 2025 happiness scores |
| `Opis Varijabli.csv` | Variable descriptions and sources, in Croatian |

## Reproducing

Requires R with `dplyr`, `ggplot2`, `corrplot` and `car`.

```r
install.packages(c("dplyr", "ggplot2", "corrplot", "car", "rmarkdown"))
rmarkdown::render("WHR_sap.Rmd")
```

The Rmd reads the CSVs from the working directory.

## Methods used

Descriptive statistics · paired t-test with visual normality assessment · multiple linear
regression · median imputation · correlation screening for multicollinearity · coefficient
standardisation for comparability · residual diagnostics · one-way ANOVA · Shapiro-Wilk test
· Pearson's chi-squared test of independence with an explicit expected-frequency check

## Sources

World Happiness Report (UN Sustainable Development Solutions Network, Gallup World Poll),
World Bank World Development Indicators, WHO Global Health Observatory, Legatum Prosperity
Index. Dataset assembled as provided for the course.
