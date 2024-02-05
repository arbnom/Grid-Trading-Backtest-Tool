import pandas as pd

def calculate_grid_values_debug(csv_path, start_time, end_time, grid_range, num_grids):
    print("Debug: Calculating grid values...")
    df = pd.read_csv(csv_path)
    df['time'] = pd.to_datetime(df['time'])
    mask = (df['time'] >= start_time) & (df['time'] <= end_time)
    df_filtered = df.loc[mask]
    
    if df_filtered.empty:
        print("Debug: No data available in the specified time range.")
        raise ValueError("No data available in the specified time range.")
    
    low, high = grid_range
    grid_size = (high - low) / num_grids
    grid_values = [low + i * grid_size for i in range(num_grids + 1)]
    
    print(f"Debug: Grid values calculated: {grid_values}")
    return grid_values

def create_price_sequence_debug(data, grid_values):
    print("Debug: Creating price sequence...")
    initial_price = data.iloc[0]['average_low_high']
    waiting_point = min(grid_values, key=lambda x: abs(x - initial_price))
    waiting_point_index = grid_values.index(waiting_point)

    sequence = []
    out_of_range = False

    for index, row in data.iterrows():
        price = row['average_low_high']

        if out_of_range and min(grid_values) <= price <= max(grid_values):
            out_of_range = False
            print(f"Debug: Price back in range at time {row['time']}, price: {price}")

        if not out_of_range:
            if waiting_point_index + 1 < len(grid_values) and price >= grid_values[waiting_point_index + 1]:
                sequence.append(1)
                waiting_point_index += 1
                print(f"Debug: Added 1 to sequence, new waiting point index: {waiting_point_index}")

            elif waiting_point_index - 1 >= 0 and price <= grid_values[waiting_point_index - 1]:
                sequence.append(-1)
                waiting_point_index -= 1
                print(f"Debug: Added -1 to sequence, new waiting point index: {waiting_point_index}")

        if price < min(grid_values) or price > max(grid_values):
            out_of_range = True
            print(f"Debug: Price out of range at time {row['time']}, price: {price}")

    print(f"Debug: Final sequence: {sequence}")
    return sequence

def calculate_required_assets_debug(grid_values, initial_price, quantity_per_grid):
    print("Debug: Calculating required assets...")
    waiting_point_index = grid_values.index(min(grid_values, key=lambda x: abs(x - initial_price)))
    required_ETH = sum(1 for i, val in enumerate(grid_values) if i > waiting_point_index) * quantity_per_grid
    required_USDC = sum(val for i, val in enumerate(grid_values) if i < waiting_point_index) * quantity_per_grid
    print(f"Debug: Required ETH: {required_ETH}, Required USDC: {required_USDC}")
    return required_ETH, required_USDC

def calculate_impermanent_loss_debug(sequence, quantity_per_grid, grid_values, initial_ETH):
    print("Debug: Calculating impermanent loss...")
    diff_between_grids = grid_values[1] - grid_values[0]
    L = len(sequence)
    S = sum(sequence)
    impermanent_loss = 0

    if S < 0:
        impermanent_loss = (quantity_per_grid * diff_between_grids * ((abs(S) - 1) * abs(S)) / 2)
    elif S > 0:
        impermanent_loss = (quantity_per_grid * diff_between_grids * ((S - 1) * S) / 2)
    
    print(f"Debug: Impermanent Loss Calculated: {impermanent_loss}")
    return impermanent_loss

def calculate_realized_profit_debug(sequence, quantity_per_grid, grid_values):
    print("Debug: Calculating realized profit...")
    diff_between_grids = grid_values[1] - grid_values[0]
    L = len(sequence)
    S = sum(sequence)
    realized_profit = quantity_per_grid * diff_between_grids * (L - abs(S)) / 2

    print(f"Debug: Realized Profit Calculated: {realized_profit}")
    return realized_profit

def calculate_end_position_assets_debug(sequence, initial_ETH, initial_USDC, quantity_per_grid, grid_values, initial_price):
    print("Debug: Calculating end position assets...")
    waiting_point_index = grid_values.index(min(grid_values, key=lambda x: abs(x - initial_price)))
    final_ETH = initial_ETH
    final_USDC = initial_USDC

    for action in sequence:
        if action == 1 and waiting_point_index + 1 < len(grid_values):
            final_ETH -= quantity_per_grid
            final_USDC += quantity_per_grid * grid_values[waiting_point_index + 1]
            waiting_point_index += 1
            print(f"Debug: Sell Order Executed. New ETH: {final_ETH}, New USDC: {final_USDC}")
        elif action == -1 and waiting_point_index - 1 >= 0:
            final_ETH += quantity_per_grid
            final_USDC -= quantity_per_grid * grid_values[waiting_point_index - 1]
            waiting_point_index -= 1
            print(f"Debug: Buy Order Executed. New ETH: {final_ETH}, New USDC: {final_USDC}")

    return final_ETH, final_USDC

def calculate_total_values_debug(initial_ETH, initial_USDC, final_ETH, final_USDC, initial_ETH_price_in_USDC, final_ETH_price_in_USDC):
    print("Debug: Calculating total values...")
    total_value_initial_USDC = (initial_ETH * initial_ETH_price_in_USDC) + initial_USDC
    total_value_final_USDC = (final_ETH * final_ETH_price_in_USDC) + final_USDC
    print(f"Debug: Total Initial Value in USDC: {total_value_initial_USDC}")
    print(f"Debug: Total Final Value in USDC: {total_value_final_USDC}")
    return total_value_initial_USDC, total_value_final_USDC

# Ensure to call these debug functions in your main execution block instead of the original functions.

# Parameters
csv_path = "path_to_the_csv_file.csv"
start_time = '2024-01-25T00:00:00Z'
end_time = '2024-01-26T00:00:00Z'
grid_range = (2200, 2300)
num_grids = 10
quantity_per_grid = 1

# Execution
# Calculate grid values
grid_values_debug = calculate_grid_values_debug(csv_path, start_time, end_time, grid_range, num_grids)

# Load the filtered data
filtered_data_debug = pd.read_csv(csv_path)
filtered_data_debug['time'] = pd.to_datetime(filtered_data_debug['time'])
filtered_data_debug = filtered_data_debug[(filtered_data_debug['time'] >= start_time) & (filtered_data_debug['time'] <= end_time)]

# Debugging sequence generation with initial assets calculation
initial_price_debug = filtered_data_debug.iloc[0]['average_low_high']
print(f"Initial ETH Price in USDC: {initial_price_debug}")
price_sequence_debug = create_price_sequence_debug(filtered_data_debug, grid_values_debug)

# Calculate required assets with debug
initial_ETH_debug, initial_USDC_debug = calculate_required_assets_debug(grid_values_debug, initial_price_debug, quantity_per_grid)
print(f"Required ETH: {initial_ETH_debug}, Required USDC: {initial_USDC_debug}")

# Calculate impermanent loss with debug
impermanent_loss_debug = calculate_impermanent_loss_debug(price_sequence_debug, quantity_per_grid, grid_values_debug, initial_ETH_debug)

# Calculate realized profit with debug
realized_profit_debug = calculate_realized_profit_debug(price_sequence_debug, quantity_per_grid, grid_values_debug)
print(f"Realized Profit in USDC: {realized_profit_debug}")

# Calculate final position assets with debug
final_ETH_debug, final_USDC_debug = calculate_end_position_assets_debug(price_sequence_debug, initial_ETH_debug, initial_USDC_debug, quantity_per_grid, grid_values_debug, initial_price_debug)
print(f"Final ETH: {final_ETH_debug}, Final USDC: {final_USDC_debug}")

# Get the final ETH price in USDC directly from the end time data
final_ETH_price_in_USDC_debug = filtered_data_debug[filtered_data_debug['time'] == end_time]['average_low_high'].iloc[-1]
print(f"Final ETH Price in USDC: {final_ETH_price_in_USDC_debug}")

# Calculate total values with debug
total_value_initial_USDC_debug, total_value_final_USDC_debug = calculate_total_values_debug(initial_ETH_debug, initial_USDC_debug, final_ETH_debug, final_USDC_debug, initial_price_debug, final_ETH_price_in_USDC_debug)
print(f"Total Initial Value in USDC: {total_value_initial_USDC_debug}")
print(f"Total Final Value in USDC: {total_value_final_USDC_debug}")

# Calculate final result in USDC
final_result_in_USDC_debug = total_value_final_USDC_debug - total_value_initial_USDC_debug
print(f"Final Result in USDC: {final_result_in_USDC_debug}")
