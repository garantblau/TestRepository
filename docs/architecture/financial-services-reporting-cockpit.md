# Financial Services Reporting Cockpit for Automotive Captives

## 1. Purpose and Scope

This document defines a reporting target model for:

- executive reporting (monthly, one-page group view)
- market steering cockpit (weekly/daily operational view by country and region)

The scope is a financial services captive of an automotive manufacturer covering financing, leasing, and related services.

## 2. Top Executive Dashboard (monthly, one-page)

### 2.1 Management summary (top section)

Use five core KPIs with traffic-light status and deltas against plan, prior year, and prior month:

| KPI | Definition | Delta views |
| --- | --- | --- |
| New business volume | Total value of newly originated contracts in period | vs plan, vs prior year, vs prior month |
| Profit before tax | Result before tax for financial services operations | vs plan, vs prior year, vs prior month |
| Cost-income ratio | Operating cost divided by operating income | vs plan, vs prior year, vs prior month |
| 90+ DPD rate | Share of contracts more than 90 days past due | vs plan, vs prior year, vs prior month |
| Retention at contract end | Share of customers taking a follow-up contract or product | vs plan, vs prior year, vs prior month |

### 2.2 Growth and profitability

KPIs:

- new business volume
- contract portfolio outstanding
- net interest margin (NIM)
- earnings by product line (financing, leasing, insurance/services)
- return on equity (RoE)

Visual standards:

- rolling 12-month trend
- product mix split (financing/leasing/insurance)

### 2.3 Risk and portfolio quality

KPIs:

- 30/60/90+ DPD rates
- non-performing loan (NPL) ratio
- expected credit loss (ECL) ratio
- write-off rate
- recovery rate

Visual standards:

- vintage view
- bucket migration view

### 2.4 Leasing and residual value risk

KPIs:

- residual value deviation (booked vs realized)
- remarketing margin
- days to sale (inventory standing days)
- realization rate

Visual standards:

- market value vs calculated residual value

### 2.5 Capital, liquidity, and refinancing

KPIs:

- funding cost
- funding mix
- ALM maturity gap
- liquidity buffer

Visual standards:

- maturity ladder
- interest and spread trend

### 2.6 Customer and loyalty

KPIs:

- contract renewal rate
- cross-sell rate
- net promoter score (NPS)
- complaint rate

Visual standards:

- end-to-end funnel from contract end to follow-up contract

## 3. Traffic-light logic

Default status logic for KPI steering:

- green: target achieved or exceeded
- yellow: deviation within 5-10% (by KPI threshold)
- red: deviation above 10% or negative trend across 3 consecutive months

## 4. Market Steering Cockpit (weekly/daily, by country and region)

### 4.1 Sales performance by market/channel/dealer

KPIs:

- penetration rate
- lead-to-contract conversion
- payout turnaround time
- dealer ranking

Drilldown path:

- country -> region -> dealer -> product

### 4.2 Pricing and product mix

KPIs:

- take rate by product
- average APR / money factor
- subsidy ratio
- margin per deal

Primary use:

- fast campaign and price-list steering

### 4.3 Origination efficiency

KPIs:

- time-to-yes
- straight-through-processing (STP) rate
- decline reason distribution
- documentation completeness

Primary use:

- bottleneck detection in credit origination

### 4.4 Collections early warning

KPIs:

- early arrears (1-29 DPD)
- cure rate
- promise-to-pay hit rate
- roll rate

Primary use:

- segment and dealer level action steering

### 4.5 End-of-term and retention

KPIs:

- return rate
- follow-up contract rate
- residual value deviation on returned units

Primary use:

- protection of existing customer base and remarketing margin

## 5. Initial target thresholds (example values)

Thresholds are market-specific and must be calibrated per country:

| KPI | Green | Yellow | Red |
| --- | --- | --- | --- |
| Penetration rate | > 35% | 30-35% | < 30% |
| Conversion rate | > 25% | 20-25% | < 20% |
| Time-to-yes | < 10 min | 10-30 min | > 30 min |
| 90+ DPD rate | < 1.5% | 1.5-2.0% | > 2.0% |
| End-of-term retention | > 55% | 45-55% | < 45% |

## 6. Prioritized data sources for MVP (8-12 weeks)

1. contract and portfolio systems (loan/leasing)
2. CRM plus lead and campaign platforms
3. dealer and DMS data
4. collections systems
5. finance general ledger plus funding and treasury data
6. external used-car market and residual value indices
7. external credit bureau and macroeconomic data

## 7. Implementation notes

- standardize KPI definitions in one governed KPI catalog before dashboard build
- define one data owner per KPI domain
- align cut-off dates for financial, risk, and operational KPIs
- include drilldown-ready dimensions (market, channel, dealer, product, vintage, segment)
