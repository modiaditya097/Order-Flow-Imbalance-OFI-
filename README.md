# OFI Feature Computation Notebook

This Jupyter Notebook processes event-level LOB snapshots, aggregates them into regular time intervals (with an option to implement without aggreation), and outputs various OFI-related features (best-level OFI, multi-level OFI sum, integrated OFI via PCA, and cross-asset OFI). 

## Order Flow Imbalance (OFI)

Order Flow Imbalance (OFI) is a market microstructure indicator that measures the net difference between buying pressure and selling pressure in the order book.  Intuitively, if there is more liquidity (depth) on the bid side than the ask side, price is more likely to move upward, and vice versa.  By quantifying this supply/demand imbalance, OFI provides insight into short-term price dynamics.  

Empirically, OFI has been shown to correlate with price changes. For example, Cont *et al.* demonstrate that, over short intervals, price movements are largely driven by the order flow imbalance at the best bid and ask. More generally, OFI models “quantify the imbalance between buying and selling pressure in financial markets”, helping predict short-term price movements. These properties make the OFI feature useful as a quantitative signal in market microstructure analysis.

## Input Data Format

The notebook expects a CSV file containing LOB snapshots at each event (e.g., each row is a snapshot after a trade or order book update). The file should have at least the following columns:  
- `ts_event`: Timestamp of the event (datetime format).  
- `symbol`: Asset symbol (e.g., stock ticker).  
- Bid price and size columns for each depth level (e.g. `bid_px_00, bid_sz_00, bid_px_01, bid_sz_01, ..., bid_px_09, bid_sz_09`).  
- Ask price and size columns (e.g. `ask_px_00, ask_sz_00, ask_px_01, ask_sz_01, ..., ask_px_09, ask_sz_09`).  

Set `BOOK_DEPTH` in the notebook to match the number of levels in your data (for example, 10 if you have levels 00–09). The data should be sorted by `symbol` and `ts_event`; the notebook does this automatically after loading.

## Usage

1. **Set parameters** at the top of the notebook:
   - `CSV_FILE`: Path to your CSV data file.  
   - `TIME_INTERVAL`: Time bin size (e.g., `'1S'` for 1 second, `'5S'`, `'1Min'`, etc.).  
   - `BOOK_DEPTH`: Number of LOB levels present in the data.  

3. **Run all cells** in order. The notebook workflow:
    - Loads the CSV and sorts by symbol and timestamp.  
    - For each symbol, computes event-level OFI at each book level with `compute_event_ofi_one_stock`.  
    - Aggregates the OFI values into the specified time intervals (`time_bin`).  
    - Computes additional features:  
      - **Best_Level_OFI**: OFI at the best bid/ask (level 1).  
      - **Multi_Level_Sum**: Sum of OFI across all levels.  
      - **Integrated_OFI**: A single OFI value (per symbol per bin) obtained by PCA on the multi-level OFIs (captures the main imbalance direction).  
      - **Cross_Asset_OFI**: For each symbol, the sum of other symbols’ integrated OFI at the same time (market-wide pressure).  

After running, the final result is a pandas DataFrame named `agg`, with one row per time bin per symbol. Columns include:  
```
time_bin, symbol, Best_Level_OFI, Multi_Level_Sum, Integrated_OFI, Cross_Asset_OFI, OFI_L1, ..., OFI_L<BOOK_DEPTH>
```
You can save this DataFrame (e.g. `agg.to_csv('ofi_features.csv')`) or use it for further analysis.

## Further Exploration

- **Adjust parameters**: Try different `TIME_INTERVAL` or `BOOK_DEPTH` settings to see how the OFI features change.    
- **Broaden assets**: Run the notebook on additional symbols or asset classes and examine cross-asset imbalances.  
- **Backtesting**: Use the computed OFI features in a trading strategy or machine-learning model.  
- **Real-time deployment**: Adapt the logic for live data feeds to compute OFI on the fly.  