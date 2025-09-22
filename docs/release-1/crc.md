# CRC Report — Currency Exchange System

## Currency
**Type:** Domain entity  
**Responsibilities:**
- Hold currency metadata (code/name, denominations, count of denominations).
- Track financial state: starting balance, current balance, critical minimum.
- Store pricing relative to local currency (buy_to_loc, sell_to_loc).
**Collaborators:**
- `CurrencyManager` (reads rates and balances during conversion and checks).
- `Utils` module (initialization, persistence of balances/denominations, I/O).

## Transaction
**Type:** Domain value object / record  
**Responsibilities:**
- Capture a single exchange operation: id, date/time, currency pair, amounts.
- Persist exchange details to CSV and support receipt/report generation.
**Collaborators:**
- `Utils` module (CSV logging, listing, searching, daily summaries, receipts).
- `Main` (creates and passes to logging/receipt generation).

## CurrencyManager
**Type:** Service  
**Responsibilities:**
- Convert an amount from currency A to currency B via Local currency:
  - Compute `LOC_in = amt_from * buy_to_loc[from]`.
  - Compute `amt_to = LOC_in / sell_to_loc[to]`.
  - Surface `rate_from_loc`, `rate_to_loc`, and `profit_delta_loc` (delta between value in and cost).
- Check reserves against `critical_min` and print alerts.
**Collaborators:**
- `Currency` (reads buy/sell rates, balances, critical minima).
- `Utils` (for globals like `currencies`, and printing).  
- `Main` (invokes conversions and checks).

## Utils (module)
**Type:** Infrastructure / Facade helpers  
**Responsibilities:**
- Initialize defaults and constants (currency names, denominations).
- User input helpers: `ask_int`, `ask_double`, `clear_input`.
- Date handling and filenames for per-day CSV.
- CSV persistence:
  - Ensure headers, append transactions, list/search by id.
  - Summaries: `csv_sum_profit_for_date`, `generate_daily_summary`.
- Receipt generation for a `Transaction`.
- State persistence for `last_transaction_id`; `shutdown()` on exit.
**Collaborators:**
- `Transaction` (serialize/deserialize/log).
- `Currency` (access global `currencies`, denominations, rates).
- `Main` (drives workflows).

## Main (program)
**Type:** Application entry point / Controller  
**Responsibilities:**
- Present CLI menu and orchestrate scenarios: exchange, show rates/balances, management actions, EOD report, manual CSV entries.
- Gather inputs and validate (delegates to `Utils`).
- Update global state (`currencies`, rates, balances) as needed.
- Use `CurrencyManager` to execute conversions and check critical reserves.
**Collaborators:**
- `CurrencyManager` (business operations).
- `Utils` (I/O, persistence).
- `Currency` & `Transaction` (domain data).

---

## Notes and Assumptions
- All conversions are performed **via local currency (LOC)** to avoid triangular arbitrage.
- Profit for a conversion is tracked as the difference between value-in-LOC and cost-in-LOC after fulfilling the target amount.
- Partial exchanges are supported in `main.cpp` (client may take part in target currency and receive remainder in LOC).
- Global arrays/vectors (`currencies`, denomination tables, names) are provided by `utils.*` and referenced by both `main.cpp` and `currency_manager.*`.
