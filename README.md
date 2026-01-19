# real-estate-investment-analysis

# Project Objective 

This project builds a residential real estate investment model that combines traditional pro-forma underwriting with data driven risk analysis. 

The goal is to move beyond single number return estimates and instead quantify return distributions, downside risk, and financing structure trade offs that matter to investors and decision makers. 

# Data Sources 
- ABS CPI Rents (Australia) was used to derive historical year on year (YoY) rent growth and calibrate Bear/Base/Bull rent growth assumptions (data/raw/abs_cpi_rents.csv)
- RBA Cash Rate Target was used to construct an interest rate series, resampled into monthly, to estimate monthly rate changes (data/raw/rba_cash_rate.xlsx)

# Financial Model Overview 
The excel model is structured in a pro-forma that produces: 
- Monthly rent income 
- Operating expenses 
- Equity cash flows and exit proceeds (sale price, selling costs, loan balance at exit) 
- Return metrics: equity IRR (XIRR), equity multiple, total profit 
- A determinisitic sensitivity table (capital growth and interest rate) 

This mirrors how investment teams underwrite deals, but is designed so key assumptions can be swapped from deterministic inputs into stochastic drivers for simulation. 

# Scenario Framework 
The underwriting assumptions are grounded in observed distributions rather than picked numbers: 
- Rent growth (bear/base/bull) is calibrated using historical percentiles 

Bear = 10th percentile

Base = 50th percentile (median) 

Bull = 90th percentile 

- Vacancy rate (bear/base/bull) inputs exists in excel, and the python workflow extends this into a simulated vacancy process for stress testing NOI 
- Interest rates are modelled as monthly changes around an estimated volatility that is bounded to avoid unrealistic jumps 

This scenario framework feeds into two workflows: 
1. Deterministic: classic sensitivity and scenario toggles in Excel 
2. Probabilities: Monte Carlo distributions 

# Statistical Analysis 
Notebook: notebooks/01_eda_rent_vacancy

Key analysis performed: 
- Rent growth distribution (histograms and summary statistics) 
- Percentile based scenario callibration (bear/base/bull)
- Tail risk framing: making downside explicit instead of it being implied 
- Interest rate processing: converting daily rates to monthly resampling and aligning with rent series dates 
- Correlation analysis: rent growth vs cash rate with correlation heatmap 

This phase translates macro and market history into inputs that can be defended rather than it being only assumption driven. 

# Monte Carlo Simulation 
Notebook: notebooks/02_random_variables_setup 

Deterministic assumptions that were initially in the excel model are converted into random variables with: 
- Central tendency (mean) 
- Dispersion (standard deviation) 
- Bounds (floors/ceilings)

Simulation engine (5,000 runs) constructs: 
- Annual rent growth draws 
- Vacancy draws 
- Interest rate change draws 
- Cash flows after debt and exit proceeds 
- Equity IRR distribution (annualised) 

The simulation outputs are then exported back to Excel so that a decision maker can work in a familiar interface while still benefitting from the probabilistic results. 

# Takeaways 
A major risk in underwriting is treating revenue drivers and financing drivers as independent while in reality, macro conditions often link them. 

In this project, historical analysis showed a strong positive correlation between rent growth and the cash rate of the sample period. 

This means that higher rates can coincide with stronger rent growth (affordability pressure pushing demands into rental properties instead). 

# Key Investment Insights
1. Returns are a distribution, not a point estimate 

The Monte Carlo results show a meaningful range of outcomes around the base-case IRR. This makes downside risk visible (left tail), which deterministic tables often hide. 

2. Financing structures materially changes the return profile

Comparing P&I and IO on the same simulated environment shows the following differences: 
- P&I shifts outcomes toward higher long-run equity IRR because amortisation reduces exit leverage (force equity builds) 
- IO improves short term cash flow, but produces a weaker long term distribution (there is more dependency on sale price/exit)

3. Downside is not hypothetical, it is measurable. 

By computing lower percentiles (5th and 10th), and probabilities, the model becomes applicable for risk adjusted decision making and not just return chasing. 

4. Deterministic sensitivies and monte carlo simulation complement each other. 

The excel sensitivity table shows where the model breaks under assumptions and monte carlo quantifies how likely those breakpoints are, given plausible distributions. 


# Challenges Encountered 

- Messy real world datasets: 

The ABS export contained mixed formatting and footnotes, which required careful coercion and parsing error handling during date time conversion 

- Frequency alignment problems: 

Rent growth data was recorded at month start, whil cash rate resampling produced month end timestamps. The workflow aligned the series to avoid false correlations caused by timestamp mismatch. 

- Vacancy data constraints: 

A full natoinal vacancy time series was not publicly available and accesible in raw form. Rather than dropping the feature, a synthetic vacancy process was built to support stress testing and monte carlo simulation on NOI, while keeping the model transparent. 

- Making excel and python "talk" to each other: 

Outputs were first exported into excel ready files so the financial model remains usable for stakeholders who avoid notebooks but still want probabilisitc insights produced by the python layer of the project. 