# The Financial Analyst — AI Research Agent Package

**Version:** 1.0 | **Level:** Intermediate | **Setup Time:** 30-45 minutes

Institutional-grade financial research delivered every morning. Configure your sectors and tickers, wake up to stock analysis, earnings tracking, and macro briefings. Built from real financial research workflows including Raoul Pal-style macro synthesis.

---

## What's In This Package

- `ROLE.md` for your financial research agent
- Daily stock research cron
- Earnings tracker template
- Macro briefing framework (AI + crypto + traditional markets)
- Sector configuration guide
- Portfolio watchlist template
- Source ranking guide

---

## ROLE.md — Financial Analyst Agent

Copy to `agents/finance/ROLE.md`:

```markdown
# ROLE.md — Financial Analyst

**Role:** Financial Research Agent
**Model:** Claude Sonnet
**Channel:** Cron-driven + sub-agent

## Responsibilities
- Daily stock and crypto research
- Macro briefings (Fed policy, rates, inflation, global)
- Earnings tracking and analysis
- Sector monitoring
- Portfolio watchlist management

## Research Standards
- Cite all sources with dates
- Distinguish: FACT vs ANALYSIS vs SPECULATION
- Flag data that's older than 7 days
- Note conflicting signals — don't just pick the bullish view
- Include bear case in every analysis

## Output Format
- Executive summary first (3 sentences max)
- Key data points (numbers, not vibes)
- Bull case / Bear case
- Actionable takeaway (if any)

## Constraints
- NEVER give buy/sell recommendations (research only)
- Flag when data is stale
- No financial advice — research and synthesis only
- Always include confidence level: HIGH/MEDIUM/LOW
```

---

## Portfolio Watchlist Configuration

Create `finance/watchlist.md`:

```markdown
# Portfolio Watchlist

## Core Holdings (Track Daily)
- AAPL — Apple
- MSFT — Microsoft
- [Add your holdings]

## Watch List (Track Weekly)
- [Tickers you're monitoring but don't own]

## Crypto Holdings
- BTC — Bitcoin
- ETH — Ethereum
- [Add your crypto holdings]

## Sector ETFs to Monitor
- SPY — S&P 500
- QQQ — NASDAQ 100
- XLK — Technology
- XLF — Financials
- [Add relevant sectors]

## Earnings Calendar
Track these companies for upcoming earnings:
- [Company 1] — next earnings: [date]
- [Company 2] — next earnings: [date]
```

---

## Sector Configuration

Create `finance/sectors.md`:

```markdown
# Sector Configuration

## Primary Focus
- [Your primary sector — e.g., Technology, Healthcare, Crypto]

## Macro Indicators to Track
- Fed Funds Rate (current + last change)
- CPI (inflation) — last reading + trend
- 10Y Treasury Yield
- DXY (Dollar Index)
- VIX (Fear Index)

## Crypto Indicators
- Bitcoin dominance
- Total market cap
- Fear & Greed Index
- ETF flows (if applicable)

## Earnings to Watch
Q1 2024 (Jan-Mar), Q2 (Apr-Jun), Q3 (Jul-Sep), Q4 (Oct-Dec)
[Update quarterly with upcoming earnings dates]

## Key Events Calendar
- [Fed meeting dates]
- [Economic data release dates: CPI, NFP, PCE]
- [Earnings dates for your holdings]
```

---

## Daily Stock Research Cron

Schedule: Weekdays at 6:00 AM (`0 6 * * 1-5`)

### Morning Brief Prompt

```
Financial morning brief. Read finance/watchlist.md and finance/sectors.md.

Deliver a pre-market briefing:

1. **Macro Pulse** (2-3 sentences)
   - Overnight futures movement
   - Any major news (Fed, economic data, geopolitical)
   - Dollar, rates, VIX status

2. **Watchlist Check** (1-2 sentences per ticker)
   - Price vs. yesterday close
   - Any news specific to this company
   - Notable chart level (52-week high/low proximity)

3. **Today's Key Events**
   - Any earnings today
   - Economic data releases
   - Fed speakers

4. **Top Trade Setup** (if applicable)
   - One ticker showing a notable setup
   - Data-backed, not a tip

Keep total under 400 words. This is pre-market — I need it fast.
```

### LaunchAgent plist

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE plist PUBLIC "-//Apple//DTD PLIST 1.0//EN" "http://www.apple.com/DTDs/PropertyList-1.0.dtd">
<plist version="1.0">
<dict>
    <key>Label</key>
    <string>com.openclaw.financial-brief</string>
    <key>ProgramArguments</key>
    <array>
        <string>/usr/local/bin/openclaw</string>
        <string>cron</string>
        <string>--message</string>
        <string>Run financial morning brief per finance/watchlist.md and finance/sectors.md</string>
    </array>
    <key>StartCalendarInterval</key>
    <array>
        <dict>
            <key>Hour</key>
            <integer>6</integer>
            <key>Minute</key>
            <integer>0</integer>
            <key>Weekday</key>
            <integer>1</integer>
        </dict>
        <dict>
            <key>Hour</key>
            <integer>6</integer>
            <key>Minute</key>
            <integer>0</integer>
            <key>Weekday</key>
            <integer>2</integer>
        </dict>
        <dict>
            <key>Hour</key>
            <integer>6</integer>
            <key>Minute</key>
            <integer>0</integer>
            <key>Weekday</key>
            <integer>3</integer>
        </dict>
        <dict>
            <key>Hour</key>
            <integer>6</integer>
            <key>Minute</key>
            <integer>0</integer>
            <key>Weekday</key>
            <integer>4</integer>
        </dict>
        <dict>
            <key>Hour</key>
            <integer>6</integer>
            <key>Minute</key>
            <integer>0</integer>
            <key>Weekday</key>
            <integer>5</integer>
        </dict>
    </array>
    <key>StandardOutPath</key>
    <string>/tmp/financial-brief.log</string>
    <key>StandardErrorPath</key>
    <string>/tmp/financial-brief.err</string>
</dict>
</plist>
```

---

## Macro Briefing Framework

Use this for weekly macro synthesis. Inspired by institutional macro analysis:

### Weekly Macro Prompt

```
Weekly macro synthesis. Research the current state of:

**Monetary Policy**
- Current Fed Funds Rate
- Most recent FOMC statement key language
- Forward guidance and market expectations (CME FedWatch data)
- Fed balance sheet trend

**Inflation**
- Latest CPI and core CPI readings
- PCE (the Fed's preferred measure)
- Trend: accelerating / decelerating / stable
- Biggest contributors (shelter, energy, food)

**Employment**
- Last NFP number
- Unemployment rate and U-6 (broad unemployment)
- Wage growth trend

**Growth**
- Latest GDP reading
- PMI (manufacturing + services)
- Consumer spending trends

**Global**
- European Central Bank stance
- China growth signals
- Emerging markets stress (dollar pressure)
- Geopolitical risks to markets

**Crypto Macro**
- Bitcoin vs. risk assets correlation
- ETF inflow/outflow data
- Regulatory environment (US, EU)

**Synthesis**
After covering all sections:
1. Overall macro regime: RISK-ON / RISK-OFF / TRANSITIONAL
2. Biggest tail risk
3. One sector likely to outperform in current environment
4. Confidence: HIGH/MEDIUM/LOW

Write to finance/macro/[DATE].md
```

---

## Earnings Tracker

### Earnings Analysis Prompt

When a company in your watchlist reports:

```
Analyze earnings for [COMPANY] ([TICKER]):

Fetch and analyze:
1. **Headline numbers** — EPS vs estimate, Revenue vs estimate
2. **Key metrics** — company-specific KPIs (MAUs, ARR, margins, etc.)
3. **Guidance** — what did management say about next quarter/year?
4. **Conference call highlights** — tone, specific comments on demand/competition
5. **Initial market reaction** — AH price movement and why

Assessment:
- Beat/Miss on: EPS / Revenue / Guidance / Sentiment
- Overall: POSITIVE / NEUTRAL / NEGATIVE
- Key risks mentioned
- Analyst reaction (if available)

Write to finance/earnings/[ticker]-[date].md
```

### Earnings Calendar Prompt

Run weekly on Sundays:

```
Build earnings calendar for the next 2 weeks.

Check upcoming earnings for all tickers in finance/watchlist.md.

Format:
| Date | Company | Ticker | Time (BMO/AMC) | EPS Estimate | Revenue Estimate |
|------|---------|--------|----------------|--------------|------------------|

Also flag:
- Any that report this week (HIGH PRIORITY)
- Sector trends — if multiple similar companies report same week, note it

Update finance/sectors.md earnings calendar section.
```

---

## Source Ranking

Quality tiers for financial research:

**Tier 1 — High reliability (use as primary sources)**
- SEC EDGAR — official filings (10-K, 10-Q, 8-K)
- Federal Reserve releases (FRED, federalreserve.gov)
- Bureau of Labor Statistics (bls.gov)
- Bureau of Economic Analysis (bea.gov)
- CME Group (FedWatch tool)

**Tier 2 — Good synthesis (use for context)**
- Bloomberg (if you have access)
- WSJ, Financial Times
- Reuters
- Yahoo Finance (for price data)
- Earnings transcripts (seekingalpha.com, earningswhispers.com)

**Tier 3 — Use with caution**
- Twitter/X financial commentary (follow official company accounts)
- Reddit (r/investing, r/wallstreetbets) — sentiment only
- Analyst reports (note the bank and potential conflicts)

**Red flags**
- No date on article
- No sources cited
- Price targets without analysis
- Articles with affiliate links to brokers

---

## Troubleshooting

**Morning brief has outdated prices**
- Market data via web search has variable freshness
- For real-time data, you'll need a Financial Datasets API key or Yahoo Finance API
- Specify: "Search for current price as of today" in your prompts

**Earnings data not found**
- Try fetching the IR page directly: `https://ir.[company].com`
- SEC EDGAR: `https://www.sec.gov/cgi-bin/browse-edgar?action=getcompany&CIK=[ticker]&type=8-K`
- Earnings transcripts: `https://seekingalpha.com/symbol/[TICKER]/earnings`

**Macro brief is too US-centric**
- Add explicit global sections to your prompt
- Specify: "Include European and Asian market context"
- Follow central bank calendars for ECB, BOJ, BOE

**Agent giving financial advice instead of research**
- Add to the top of your ROLE.md: "This is research only. Never say 'buy' or 'sell'."
- Use phrases like "shows bullish signals" not "you should buy"
- Configure explicit output format: bull/bear/neutral with data, not recommendations

---

*Built on OpenClaw. Requires OpenClaw installed and configured. For educational and research purposes only — not financial advice.*
