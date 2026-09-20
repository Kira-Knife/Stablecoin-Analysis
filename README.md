# Dollar-Pegged Stablecoins and US Monetary Policy Shocks

**Do USDT and USDC respond to Fed policy surprises - and do they respond differently?**

An empirical study of how the two largest dollar stablecoins react to identified US monetary policy shocks, 2021-2026. Event-study regressions, local projections and a VAR on monthly supply and on-chain transaction volume, using the Jarociński-Karadi (2020) high-frequency shock series.

![Python](https://img.shields.io/badge/python-3.10%2B-blue)
![Jupyter](https://img.shields.io/badge/notebook-Jupyter-orange)
![statsmodels](https://img.shields.io/badge/statsmodels-VAR%20%7C%20OLS%20%7C%20LP-lightgrey)
![License: MIT](https://img.shields.io/badge/license-MIT-green)

---

## Question

Stablecoins are private dollar liabilities that pay no interest. When the Fed tightens, the opportunity cost of holding them rises, so supply and usage should contract; when the Fed signals good news about the economy (an "information" shock), the sign may flip. USDT and USDC differ in reserve composition, user base and regulatory exposure, which could make their responses asymmetric.

This repository tests both hypotheses on identified shocks rather than on policy-rate changes, which are largely anticipated and confound policy with the news that caused it.

## Key findings

Event-study regressions of monthly changes in supply and adjusted on-chain volume on the Jarociński–Karadi **pure monetary policy shock (MP)** and **central bank information shock (CBI)**, January 2021 - March 2026, 62 monthly observations, HC1 robust standard errors.

| Dependent variable (Δ, bn USD) | MP shock | CBI shock | R² |
|---|---:|---:|---:|
| Supply USDT | −4.69 \* (2.60) | 16.81 (10.32) | 0.06 |
| Supply USDC | −8.71 \*\* (3.68) | 21.67 \* (13.10) | 0.10 |
| Adjusted volume USDT | −383.0 \* (196.2) | −1,331.6 \*\* (648.6) | 0.06 |
| Adjusted volume USDC | −479.0 (414.1) | −2,664.9 (1,780.0) | 0.00 |

\*\*\* p<0.01, \*\* p<0.05, \* p<0.1. Standard errors in parentheses.

- **A contractionary monetary policy surprise reduces stablecoin supply.** The effect is significant at 5% for USDC and at 10% for USDT; it survives the switch to median-rotation shocks (USDC p=0.05, USDT volume p=0.05) and there are no influential outlier months.
- **Information shocks work in the opposite direction on supply** (positive point estimates, USDC significant at 10%), consistent with the two-shock decomposition: good news about the economy raises demand for on-chain dollars even as rates rise.
- **USDC's point estimate is roughly twice USDT's**, but the difference is not statistically significant (p=0.38 for supply). The asymmetry hypothesis is suggestive, not established, on this sample.
- **Local projections** (Newey-West, 4 lags, horizon 12) put the peak supply response at **h=3 months**: about -4.8 bn USD for USDT and -9.5 bn for USDC per unit shock, again with no significant USDT–USDC difference.
- **VIX co-moves strongly with supply changes** (ρ ≈ -0.64 for USDT, -0.56 for USDC): risk-off episodes are the single largest correlate of stablecoin contraction in the sample, which is why the shock-based identification matters.

![Stablecoin supply, 2021–2026](figures/fig1_supply_trends.png)

![Jarociński–Karadi monetary policy shocks](figures/fig3_mp_shocks.png)

## Data

| Series | Source | Frequency | Coverage |
|---|---|---|---|
| Monetary policy (MP) and central bank information (CBI) shocks; poor-man's and median-rotation variants | Jarociński & Karadi (2020), updated series | Monthly | Jan 2021 - Mar 2026 |
| USDT, USDC circulating supply (USD) | <!-- TODO: confirm — Visa Onchain Analytics / Allium dashboard export --> Onchain analytics dashboard export | Monthly | Jan 2021 - Mar 2026 |
| USDT, USDC adjusted transaction volume (USD) — filtered for bots and inorganic activity | <!-- TODO: confirm source --> Onchain analytics dashboard export | Monthly | Jan 2021 - Mar 2026 |
| USDT, USDC market capitalisation | CoinMarketCap historical data | Daily | 2025 - Mar 2026 |
| CBOE Volatility Index (VIXCLS) | FRED, Federal Reserve Bank of St. Louis | Daily - monthly | 2021 - 2026 |

The merged monthly panel has 63 observations (62 after first-differencing). Supply gaps before 2022 are linearly interpolated. Raw data files are **not** included in the repository because of provider licence terms; the notebook expects them in the working directory under the file names given in the first cells.

## Methods

1. **Stationarity.** ADF and KPSS on levels and first differences. Supply and volume are I(1); the MP shock is I(0) by construction; all regressions use monthly differences of the outcomes and shocks in levels.
2. **Event study.** OLS of Δsupply and Δvolume on MP and CBI shocks, HC1 standard errors, Durbin–Watson and Jarque–Bera diagnostics. Robustness: median-rotation shocks (MP_median, CBI_median); re-estimation without |standardised residual| > 2 months.
3. **Cross-coin comparison.** Wald-type test of equality of USDT and USDC coefficients.
4. **Local projections** (Jordà 2005). Horizons 0-12, four lags of the outcome and of each shock, HAC standard errors, 68% and 90% bands. Run separately for MP and CBI shocks.
5. **VAR.** Five-variable monthly VAR (ΔSupply USDT, ΔSupply USDC, ΔVolume USDT, ΔVolume USDC, MP), lag order by AIC, bootstrap IRF bands (500 replications), robustness to lag order and Cholesky ordering. Given 62 observations the VAR is imprecise and is reported as an exploratory complement to the local projections, not as the main result.

## Repository layout

```
.
├── main_analysis.ipynb        # full pipeline: data prep → tests → event study → LP → VAR
├── figures/                   # figures exported from the notebook
│   ├── fig1_supply_trends.png
│   ├── fig2_adjusted_volume.png
│   ├── fig3_mp_shocks.png
│   ├── fig4_cbi_shocks.png
│   ├── fig5_usdt_supply_vs_volume.png
│   ├── fig6_usdc_supply_vs_volume.png
│   ├── fig7_var_irf_mp.png
│   ├── fig8_lp_irf_mp.png
│   └── fig9_lp_irf_cbi.png
├── requirements.txt
├── LICENSE                    # MIT
└── README.md
```

The notebook writes intermediate CSVs (`final_merged_data_2021_2026.csv`, `stationarity_results.csv`, `correlation_matrix*.csv`, `table_3_3_event_study_results.csv`) and figures into `chapter_3_results/`.

## Reproducing

```bash
git clone https://github.com/Kira-Knife/Stablecoin-Analysis.git
cd Stablecoin-Analysis
pip install -r requirements.txt
jupyter notebook main_analysis.ipynb
```

The notebook was developed in a Pyodide/JupyterLite kernel, so the first cell installs `statsmodels` via `micropip`. In a standard Python environment delete or comment out the two `micropip` lines; everything else runs unchanged.

Place the input files in the repository root:

- `shocks_fed_jk_m.csv` - Jarociński–Karadi monthly shocks
- `Stablecoin Usage by Stablecoin - Breakdown by Stablecoins - Supply (USD).csv`
- `Stablecoin Usage by Stablecoin - Breakdown by Stablecoins - Adjusted Transaction Volume.csv`
- `Tether USDt_*_historical_data_coinmarketcap.csv`, `USDC_*_historical_data_coinmarketcap.csv`
- `VIXCLS.csv`

## Limitations

- 62 monthly observations is a short sample for a five-variable VAR; the local projections and event study carry the weight of the evidence.
- Shock identification is borrowed from the Jarociński–Karadi decomposition; results inherit its assumptions.
- Supply and adjusted volume come from a single analytics provider's methodology for filtering inorganic activity.
- Coefficients are in billions of USD per unit shock, not elasticities; with supply growing from ~60 bn to ~190 bn over the sample, level effects are not directly comparable across years.

## Citation

<!-- TODO: replace with the thesis / preprint reference and DOI once available -->

```
Lanina, P. (2026). Dollar-Pegged Stablecoins and US Monetary Policy Shocks:
Evidence from Identified Shocks, 2021–2026. GitHub repository,
https://github.com/Kira-Knife/Stablecoin-Analysis
```

**References.** 
Jarocinski, M. & Karadi, P. (2020). Deconstructing Monetary Policy Surprises — The Role of Information Shocks. *American Economic Journal: Macroeconomics*, 12(2), 1-43. 
Jorda, O. (2005). Estimation and Inference of Impulse Responses by Local Projections. *American Economic Review*, 95(1), 161-182.

## Author

**Polina Lanina** - independent payments researcher.
Papers as Lanina, code as [@Kira-Knife](https://github.com/Kira-Knife)  [ORCID 0000-0002-3010-6813](https://orcid.org/0000-0002-3010-6813)  [kira-knife.github.io](https://kira-knife.github.io)

Work done in a personal capacity. Licensed under MIT.
