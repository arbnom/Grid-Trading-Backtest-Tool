# Grid Trading Backtest Tool

## Description
This tool is designed to calculate the results of a grid trading strategy given specific parameters. It utilizes raw ETH/USDC data, which includes timestamps, open, high, low, close, and volume values, typically exported from TradingView. The main dataset, named `average_price_data.csv`, is derived from the raw data by averaging the high and low values for each minute.

The repository is organized into two main sections, each containing four files. The scripts in the `with_stop_loss` directory assume that a stop-loss is triggered when the price moves outside the predefined grid range, resulting in the closure of the grid trading position. Conversely, the scripts in the `without_stop_loss` directory allow the grid trading strategy to pause and then resume as the price re-enters the range, without triggering a stop loss.

## Key Components

### Grid Value Calculation (`calculate_grid_values`)
- Establishes the grid by dividing a price range into equal segments based on the user-defined number of grids, creating the grid lines within the trading range.

### Price Sequence Generation (`create_price_sequence`)
- Generates a -1 and 1 sequence representing buy and sell signals within the grid. "1" indicates a sell order executed when the price moves up crossing a grid line, and "-1" indicates a buy order executed when the price moves down crossing a grid line.

### Required Assets Calculation (`calculate_required_assets`)
- Determines the initial assets (ETH and USDC) required to set up buy and sell orders at each grid line, excluding the line nearest to the initial price to establish the starting point.

### Impermanent Loss and Realized Profit Calculation
- Calculates the theoretical impermanent loss and realized profit from the grid trading strategy's buy and sell actions, providing insight into the strategy's performance.

### End Position Assets Calculation (`calculate_end_position_assets`)
- Calculates the final ETH and USDC holdings after considering all executed buy and sell orders throughout the trading period.

### Total Value Calculation (`calculate_total_values`)
- Assesses the total value of the trading account in USDC at the beginning and end of the trading period to evaluate the grid trading strategy's overall effectiveness.

## Concepts

- **Grid Trading Strategy**: Involves placing a ladder of buy and sell orders within a defined price range to capitalize on market volatility.
- **-1,1 Sequence**: Central to understanding the grid strategy's operations over time, with "-1" indicating a buy action and "1" a sell action.

This backtest tool provides a systematic way to evaluate grid trading strategies, accommodating scenarios both with and without a stop-loss mechanism, to help users gauge potential outcomes based on historical price data.
