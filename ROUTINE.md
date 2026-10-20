# Outsmart the Open: routine instructions

These are the instructions the scheduled "Outsmart the Open" routine follows each ASX
trading morning. The routine's own prompt points here and supplies the email recipient.
Edit this file to change what the brief contains; the change applies from the next run.

---

Produce "Outsmart the Open", a daily pre-market brief for an Australian investor, in the style of a newspaper pre-market column (the AFR's "Before the Bell" is the model for tone and structure, but never use that name). It is currently early morning Sydney time (~05:30-06:30) on an ASX trading day; US and European markets have just closed overnight. The reader holds broad-market Australian and global ETFs, so keep everything at index/market level — do not discuss or recommend individual stocks or specific ETF products except where a named instrument IS the correct way to express a market (e.g. SPI 200 / ASX 200 index futures, S&P 500 e-mini futures, VIX). Individual companies may be named in the news section only where they are the news (e.g. a mega-cap earnings miss that moved an index).

CRITICAL FIRST STEP — determine the correct Sydney trading date: this task runs on a schedule fixed in UTC, and the container's default `date`/`date -u` output is UTC, NOT Sydney time. At the scheduled fire time, the Sydney calendar date is always one day ahead of the UTC date. Do NOT use any "today's date" value from system/session context for the dateline — that reflects the UTC/session date, not Sydney local time, and using it will misdate the whole brief. Instead, run this in bash before anything else: `TZ=Australia/Sydney date +"%A %d %B %Y %H:%M"` — use THAT output as the actual current Sydney date/time for the dateline, email subject, and to correctly identify which overnight session you are reporting on (the most recently completed US/European market close relative to this Sydney date/time — normally the prior Sydney calendar day's US/Europe session). If the Sydney date output ever looks inconsistent with the news you find (e.g. sources describe a different date as "today"), trust the `TZ=Australia/Sydney date` command output, not your prior assumptions or system context.

ASX HOLIDAY CHECK — next, decide whether the ASX is open today (the schedule fires every weekday, including public holidays). Check the ASX trading calendar (search "ASX trading hours holidays <year>"; asx.com.au lists them). If the ASX is closed today for a public holiday: do NOT research or send an email. Instead, follow the PUBLISHING steps below to add an index entry {"date": "<Sydney date>", "status": "closed", "closed_reason": "<holiday name>"} (no brief file), commit "ASX closed <date>: <holiday>", push, reply with one line saying so, and stop.

RETRY AT 09:00 — if, after following the sourcing and reconciliation rules, the data is too unreliable to publish a trustworthy brief (for example the US session had not closed yet, sources conflict on SPI direction or on the US index closes, or most scoreboard rows cannot be verified), do NOT send an email or publish on this first attempt. Instead: (1) schedule a retry for 09:00 Sydney time today (the ASX opens at 10:00) using the `send_later` tool (claude-code-remote MCP server) with `at` set to the RFC3339 time, with a message telling the retry to re-run this routine in full for the same Sydney date and email recipient; (2) send a short PushNotification saying the brief was deferred to 09:00 and why. On the 09:00 retry, make a fresh attempt at every source; if it still cannot be verified, send the brief anyway using "not available" for unresolved figures per the reconciliation rules (with a caveat in the intro and footnotes), and publish it as normal. Do not schedule a second retry.

Use web search and WebFetch to get actual current data. Do not rely on training knowledge for any price, level, or percentage — every number must come from a source you fetched in this run.

SOURCES (tested and known to work; try them directly with WebFetch rather than relying on search results alone):
- FNArena daily report — the best single one-page overnight scoreboard. URL pattern: https://fnarena.com/YYYY/MM/DD/the-<weekday>-report-<D>-<month>-<YYYY>/ using the Sydney date (e.g. https://fnarena.com/2026/10/12/the-monday-report-12-october-2026/). It lists SPI 200 futures, VIX, spot gold, AUD/USD, iron ore, Brent/WTI, US indices, US 10-year yield, FTSE/DAX and the prior ASX 200 close. If the URL is not found, web-search "FNArena Monday/Tuesday/... Report <date>".
- SPI 200 futures: TradingView https://www.tradingview.com/symbols/ASX24-AP1!/ ; Investing.com https://www.investing.com/indices/australia-200-futures (check which contract month is shown — a far-dated contract such as Dec can be stale/mispriced vs front month; treat with caution) ; FNArena.
- VIX: Yahoo Finance https://finance.yahoo.com/quote/%5EVIX/ ; FNArena.
- US close, sectors, yields, Brent: AP wrap (search "Wall Street" AP closing story; syndicated on cp24.com, bnnbloomberg.ca, baltimoresun.com), CNBC, Reuters via Kitco.
- Gold, AUD/USD, iron ore, ASX 200, US 10-year: Trading Economics pages (tradingeconomics.com/commodity/gold, /commodity/iron-ore, /australia/currency, /australia/stock-market, /united-states/government-bond-yield). Note these are CFD-based reference prices.
- Forward calendar: Saxo "week ahead" and Kitco/Reuters "Wall St Week Ahead".
- AFR "Before the Bell" (site:afr.com "before the bell") is a nice-to-have cross-check if accessible, but the brief MUST NOT depend on it — it is usually paywalled.
- Known NOT to work: Yahoo futures page AP=F (robots-blocked), Google Finance for SPI, Stooq (blocked), Morningstar AU (no daily global market report any more), CNBC .VIX quote page (errors), Investing.com historical-data pages (stale).

RECONCILIATION RULES:
- Never take a single-source figure at face value for SPI 200 futures; fetch at least two independent sources for the level and direction.
- You must try at least three of the listed sources for SPI 200 futures and for the VIX before ever writing "not available". "Not available" is only acceptable if every listed source has been tried and failed, or if sources disagree on DIRECTION and cannot be resolved.
- If two sources agree on direction and are within a few points / ~0.1% of each other, report the figure given by the sources that agree and footnote the other. If they agree on direction but differ materially on level, report the figure confirmed by two sources if one exists; otherwise report the direction and the range (e.g. "up roughly 0.4-0.8%, level 8,745-8,786") with a footnote — do not blank it out.
- For other figures, if two independent sources materially disagree on a figure and you can't resolve it, write "not available". Small rounding or timing differences (e.g. +1.41% vs +1.47%, a settlement vs a later snapshot) are not material: pick one, footnote it.
- Implied ASX 200 open: apply the SPI futures % change to the prior ASX 200 cash close (futures levels carry a basis premium vs cash, so do NOT subtract the cash close from the futures level). State the implied open in points and per cent.
- Before finalizing, run an internal consistency check (a past run got the SPI direction sign-flipped): SPI direction should logically match the overnight US/global risk tone. If the write-up describes a risk-off session but SPI is UP, or vice versa, stop and re-verify against a second source. Do not include a number that contradicts your own narrative without resolving the contradiction first. Likewise if the S&P 500 % change doesn't match the qualitative tone of the news, re-check before finalizing.

CONTENT — structure exactly as follows, in five numbered sections:

1. Overnight scoreboard — a table with columns: Index / Instrument | Close / Last | % Chg. Include EXACTLY these rows, in this order, and no others:
   - S&P 500
   - NASDAQ Composite
   - Dow Jones Industrial Average
   - VIX
   - US 10-year Treasury yield (level as %, change in bp)
   - AUD/USD
   - Brent crude
   - Spot gold
   - Iron ore (62% Fe, CFR China) — omit this row only if no print is available
   Do NOT include STOXX Europe 600, FTSE 100, Nikkei 225 or other regional indices in this table — the reader does not follow them. Notable European or Asian index moves belong in section 3 or 4 as prose only, and only when they are genuinely part of the story. State the currency for every price (e.g. US$93.43/bbl, US$4,520.30/oz, US$95.17/t). Metric/SI units throughout.

2. ASX 200 futures / today's set-up — the SPI 200 / ASX 200 index futures level and change, and what it implies for the ASX 200 open in points and per cent (see reconciliation rules). Note the prior ASX 200 close and % change for reference. Then a bolded "Local data/events today:" line flagging scheduled Australian data releases, RBA speeches or decisions, and notable local earnings due today.

3. What moved markets — 2-4 sentences on the dominant driver(s) of the overnight session. Name which S&P 500 sectors led and lagged, with percentages, if reported.

4. Top news — 5 to 10 items, each a short bold heading followed by a 1-2 sentence summary, ending with "Source: <linked publication name>". Prioritise items that matter to a diversified index investor: central bank policy and rates, inflation and labour data, US/China/AU macro, energy and commodity shifts, major geopolitical or trade developments, and market-structure events.

5. Coming up — one or two lines on the next significant scheduled macro event or data release, with its date.

Style: matter-of-fact, no hype, no filler, no investment advice or recommendations. If a data point cannot be verified after following the sourcing rules above, write "not available" rather than estimating — do not guess or interpolate. Where two sources disagree and you settle on one, add a short footnote under the table explaining the reconciliation. If markets were closed overnight for a US or European public holiday, say so explicitly and report what did trade. Keep the whole thing tight enough to read in about three minutes.

OUTPUT FORMAT — use this exact HTML template every day so the email looks identical from one morning to the next. Inline CSS only, no <style> block, no scripts, no images, no external assets. Fill in the content; do not redesign the layout, change the colours, fonts, spacing or structure. The house colours are #1a1a2e (header/heading/rule), #f4f4f4 (page and footer background), #ffffff (card), #dddddd (borders), #f7f7f7 (alternating table rows), #b00000 (negative), #0a7a26 (positive), #555555/#888888/#999999 (secondary text).

<table width="100%" cellpadding="0" cellspacing="0" style="background-color:#f4f4f4;padding:20px 0">
<tr><td align="center">
<table width="640" cellpadding="0" cellspacing="0" style="background-color:#ffffff;border:1px solid #dddddd">

<tr><td style="background-color:#1a1a2e;padding:20px 24px">
<span style="color:#ffffff;font-size:20px;font-weight:bold;font-family:Georgia,'Times New Roman',serif">Outsmart the Open</span><br>
<span style="color:#c9c9d6;font-size:13px">ASX Pre-Market Brief — [WEEKDAY DD MONTH YYYY], [HH:MM] AEST</span>
</td></tr>

<tr><td style="padding:16px 24px 4px 24px">
<p style="font-size:13px;color:#555555;line-height:1.5;margin:0 0 12px 0">[One or two sentences stating which overnight session is covered and any sourcing caveats.]</p>
</td></tr>

<tr><td style="padding:0 24px">
<h2 style="font-family:Georgia,'Times New Roman',serif;font-size:16px;color:#1a1a2e;border-bottom:2px solid #1a1a2e;padding-bottom:6px;margin:12px 0 10px 0">1. Overnight Scoreboard</h2>
<table width="100%" cellpadding="0" cellspacing="0" style="border-collapse:collapse;font-size:13px">
<tr style="background-color:#1a1a2e">
<td style="padding:8px 10px;color:#ffffff;font-weight:bold;border:1px solid #1a1a2e">Index / Instrument</td>
<td style="padding:8px 10px;color:#ffffff;font-weight:bold;border:1px solid #1a1a2e;text-align:right">Close / Last</td>
<td style="padding:8px 10px;color:#ffffff;font-weight:bold;border:1px solid #1a1a2e;text-align:right">% Chg</td>
</tr>
<tr><td style="padding:7px 10px;border:1px solid #dddddd">S&amp;P 500</td><td style="padding:7px 10px;border:1px solid #dddddd;text-align:right">0,000.00 pts</td><td style="padding:7px 10px;border:1px solid #dddddd;text-align:right;color:#b00000">-0.00%</td></tr>
<tr style="background-color:#f7f7f7"><td style="padding:7px 10px;border:1px solid #dddddd">NASDAQ Composite</td><td style="padding:7px 10px;border:1px solid #dddddd;text-align:right">0,000.00 pts</td><td style="padding:7px 10px;border:1px solid #dddddd;text-align:right;color:#b00000">-0.00%</td></tr>
[...continue alternating plain / #f7f7f7 rows for the remaining instruments, using color:#0a7a26 for positive changes and color:#555555 for flat/neutral...]
</table>
<p style="font-size:11px;color:#888888;margin:8px 0 0 0;line-height:1.4">[Footnotes on any reconciled or approximate figures. Omit this paragraph if there are none.]</p>
</td></tr>

<tr><td style="padding:20px 24px 0 24px">
<h2 style="font-family:Georgia,'Times New Roman',serif;font-size:16px;color:#1a1a2e;border-bottom:2px solid #1a1a2e;padding-bottom:6px;margin:0 0 10px 0">2. ASX 200 Futures / Today's Set-Up</h2>
<p style="font-size:13px;color:#222222;line-height:1.6;margin:0 0 8px 0">[Futures and implied open, prior ASX 200 close.]</p>
<p style="font-size:13px;color:#222222;line-height:1.6;margin:0 0 8px 0"><b>Local data/events today:</b> [...]</p>
</td></tr>

<tr><td style="padding:20px 24px 0 24px">
<h2 style="font-family:Georgia,'Times New Roman',serif;font-size:16px;color:#1a1a2e;border-bottom:2px solid #1a1a2e;padding-bottom:6px;margin:0 0 10px 0">3. What Moved Markets</h2>
<p style="font-size:13px;color:#222222;line-height:1.6;margin:0">[...]</p>
</td></tr>

<tr><td style="padding:20px 24px 0 24px">
<h2 style="font-family:Georgia,'Times New Roman',serif;font-size:16px;color:#1a1a2e;border-bottom:2px solid #1a1a2e;padding-bottom:6px;margin:0 0 14px 0">4. Top News</h2>
<p style="font-size:13px;color:#222222;line-height:1.6;margin:0 0 12px 0"><b>[Headline.]</b> [One or two sentences.] Source: <a href="[URL]" style="color:#1a1a2e">[Publication]</a></p>
[...one paragraph per item; the final item uses margin:0 instead of margin:0 0 12px 0...]
</td></tr>

<tr><td style="padding:20px 24px 20px 24px">
<h2 style="font-family:Georgia,'Times New Roman',serif;font-size:16px;color:#1a1a2e;border-bottom:2px solid #1a1a2e;padding-bottom:6px;margin:0 0 10px 0">5. Coming Up</h2>
<p style="font-size:13px;color:#222222;line-height:1.6;margin:0">[...]</p>
</td></tr>

<tr><td style="background-color:#f4f4f4;padding:14px 24px;border-top:1px solid #dddddd">
<p style="font-size:11px;color:#999999;margin:0;line-height:1.5">This is a factual market summary, not investment advice. All figures sourced from third-party providers on the morning of publication and cross-checked where possible. Prepared for a diversified index/ETF investor — no individual securities are recommended.</p>
</td></tr>

</table>
</td></tr>
</table>

Link directly to the source URL in each <a href> — do not wrap URLs in redirectors.

CRITICAL — how to deliver the email (a past run got this wrong): Do NOT write the email's HTML or text to any file. The only files this task may create are the JSON files described under PUBLISHING, inside the outsmart-the-open repository. Compose the final HTML directly as a string in your own reasoning/response, and pass that actual HTML markup — the literal `<table>...</table>` text — as the value of the `htmlBody` parameter in the `mcp__Gmail__send_message` call. The value of `htmlBody` must never be a file path or the name of a file; if you find yourself about to pass something like "/tmp/anything.html" as an argument value, stop — that is the exact bug to avoid, since it previously caused the recipient to receive an email whose entire body was the literal text of a file path instead of the brief. Build the HTML and plain-text versions as in-memory strings and pass them straight to `to`, `subject`, `htmlBody`, and `body`.

Send the finished brief directly as an HTML email using the mcp__Gmail__send_message tool: set `to` to the recipient address given in the routine prompt that pointed you to this file, `subject` to "Outsmart the Open – ASX Pre-Market Brief, " followed by the correct Sydney date (from the `TZ=Australia/Sydney date` command, NOT session context), pass the composed HTML string via the `htmlBody` parameter, and include a plain-text version of the same content in `body` as a fallback. PUBLISHING — after the email is sent, publish the same brief as structured data for the FolioScope app. The rmcleay/outsmart-the-open GitHub repository is checked out in this session (find it with `git rev-parse --show-toplevel` or `ls`; if it isn't present, clone https://github.com/rmcleay/outsmart-the-open). Work on branch main, `git pull` first.

1. Write briefs/YYYY/YYYY-MM-DD.json (the Sydney date) with exactly this shape. Keys are snake_case; numbers are plain JSON numbers with no thousands separators or units; percentages are in per cent (0.42 means +0.42%); use null for any figure you reported as "not available". Text fields are plain text (no HTML). The figures and wording must match the email exactly.
{
  "schema_version": 1,
  "date": "YYYY-MM-DD",
  "status": "published",
  "published_at": "<now, ISO 8601 with the Sydney offset, e.g. 2026-10-16T06:41:00+11:00; from TZ=Australia/Sydney date -Iseconds>",
  "session_covered": "<e.g. US and European sessions of Thursday 15 October>",
  "intro": "<the intro sentence(s)>",
  "scoreboard": [ {"key": "sp500", "name": "S&P 500", "value": 6712.40, "unit": "pts", "change": -0.42, "change_unit": "%"}, ... ],
  "footnotes": ["<each footnote under the table>"],
  "setup": {"spi_level": 8790, "spi_change_pct": 0.42, "implied_open": 8786, "implied_open_points": 36.7, "implied_open_pct": 0.42, "prior_close": 8749.5, "prior_close_change_pct": -0.31, "approximate": false, "summary": "<section 2 text, without the local events line>"},
  "local_events": ["<one item per scheduled event>"],
  "what_moved": "<section 3>",
  "news": [ {"headline": "...", "summary": "...", "source": "<publication>", "url": "<source URL>"} ],
  "coming_up": "<section 5>"
}
Scoreboard rows use these keys, in the table's order: sp500, nasdaq, dow, vix, us10y, audusd, brent, gold, iron_ore (omit iron_ore when its row is omitted). Units: "pts" for the three indices and the VIX; "%" for the 10-year yield (with change_unit "bp" and change in basis points); "US$" for AUD/USD; "US$/bbl", "US$/oz", "US$/t" for Brent, gold and iron ore. change_unit is "%" everywhere else. Set setup.approximate to true when the futures figure is a range or single-source; when it is a range, give the midpoint.

2. Update index.json (create it as {"schema_version": 1, "updated_at": null, "briefs": []} if missing): remove any entry with today's date, then insert at the front {"date", "status": "published", "published_at" (identical to the brief file's), "implied_open_pct", "implied_open_points", "scoreboard": {key: value for every scoreboard row with a value}, "headlines": [every news headline]}. Keep entries newest first and set updated_at to now.

3. Check both files parse (`python3 -m json.tool <file> > /dev/null`), then `git add` them, commit "Brief for YYYY-MM-DD", and push to main. If the push is rejected because the remote moved, pull with rebase and push again.

After publishing, reply with one line: the Sydney date and time used, that the email was sent, and the commit hash pushed (or why publishing failed — the email still counts as delivered). Do not paste the full brief into the chat response.
