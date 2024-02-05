import pandas as pd

def calculate_grid_values(csv_path, start_time, end_time, grid_range, num_grids):
    df = pd.read_csv(csv_path)
    df['time'] = pd.to_datetime(df['time'])
    mask = (df['time'] >= start_time) & (df['time'] <= end_time)
    df_filtered = df.loc[mask]

    if df_filtered.empty:
        raise ValueError("No data available in the specified time range.")

    low, high = grid_range
    grid_size = (high - low) / num_grids
    grid_values = [low + i * grid_size for i in range(num_grids + 1)]

    return grid_values

def create_price_sequence_debug(data, grid_values, quantity_per_grid):
    initial_price = data.iloc[0]['average_low_high']
    waiting_point = min(grid_values, key=lambda x: abs(x - initial_price))
    waiting_point_index = grid_values.index(waiting_point)
    required_ETH, required_USDC = calculate_required_assets(grid_values, initial_price, quantity_per_grid)

    print(f"Initial price: {initial_price}")
    print(f"Initial waiting point: {waiting_point}")
    print(f"Required ETH: {required_ETH}, Required USDC: {required_USDC}")

    L = []

    for index, row in data.iterrows():
        time, price = row['time'], row['average_low_high']

        # Only update the sequence and waiting point if the price moves beyond the next grid point
        if price >= grid_values[min(waiting_point_index + 1, len(grid_values) - 1)] and waiting_point != grid_values[min(waiting_point_index + 1, len(grid_values) - 1)]:
            L.append(1)
            waiting_point_index = min(waiting_point_index + 1, len(grid_values) - 1)
            waiting_point = grid_values[waiting_point_index]
            print(f"Time: {time}, Price: {price}, New waiting point (up): {waiting_point}, Sequence: 1")

        elif price <= grid_values[max(waiting_point_index - 1, 0)] and waiting_point != grid_values[max(waiting_point_index - 1, 0)]:
            L.append(-1)
            waiting_point_index = max(waiting_point_index - 1, 0)
            waiting_point = grid_values[waiting_point_index]
            print(f"Time: {time}, Price: {price}, New waiting point (down): {waiting_point}, Sequence: -1")

        if price < min(grid_values) or price > max(grid_values):
            print(f"Time: {time}, Price: {price} - Stopped due to price going outside of grid range.")
            break

    print(f"Final sequence: {L}")
    return L

def calculate_required_assets(grid_values, initial_price, quantity_per_grid):
    required_ETH = sum(1 for i in grid_values if i > initial_price) * quantity_per_grid
    required_USDC = sum(i for i in grid_values if i < initial_price) * quantity_per_grid
    return required_ETH, required_USDC

def calculate_impermanent_loss_debug(sequence, quantity_per_grid, grid_values, initial_ETH):
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
    diff_between_grids = grid_values[1] - grid_values[0]
    L = len(sequence)
    S = sum(sequence)
    realized_profit = quantity_per_grid * diff_between_grids * (L - abs(S)) / 2

    print(f"Debug: Realized Profit Calculated: {realized_profit}")
    return realized_profit

def calculate_end_position_assets_debug(sequence, initial_ETH, initial_USDC, quantity_per_grid, grid_values, initial_price):
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

def calculate_end_position_price_debug(sequence, grid_values, initial_price):
    waiting_point_index = grid_values.index(min(grid_values, key=lambda x: abs(x - initial_price)))

    for action in sequence:
        if action == 1 and waiting_point_index + 1 < len(grid_values):
            waiting_point_index += 1
        elif action == -1 and waiting_point_index - 1 >= 0:
            waiting_point_index -= 1

    final_ETH_price = grid_values[waiting_point_index]
    print(f"Debug: Final ETH Price Calculated: {final_ETH_price}")
    return final_ETH_price

def calculate_total_values_debug(initial_ETH, initial_USDC, final_ETH, final_USDC, initial_ETH_price_in_USDC, final_ETH_price_in_USDC):
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
grid_values = calculate_grid_values(csv_path, start_time, end_time, grid_range, num_grids)

# Load the filtered data
filtered_data = pd.read_csv(csv_path)
filtered_data['time'] = pd.to_datetime(filtered_data['time'])
filtered_data = filtered_data[(filtered_data['time'] >= start_time) & (filtered_data['time'] <= end_time)]

# Debugging sequence generation with initial assets calculation
initial_price = filtered_data.iloc[0]['average_low_high']
print(f"Initial ETH Price in USDC: {initial_price}")
price_sequence = create_price_sequence_debug(filtered_data, grid_values, quantity_per_grid)

# Calculate required assets with debug
initial_ETH, initial_USDC = calculate_required_assets(grid_values, initial_price, quantity_per_grid)
print(f"Required ETH: {initial_ETH}, Required USDC: {initial_USDC}")

# Calculate impermanent loss with debug
impermanent_loss = calculate_impermanent_loss_debug(price_sequence, quantity_per_grid, grid_values, initial_ETH)

# Calculate realized profit with debug
realized_profit = calculate_realized_profit_debug(price_sequence, quantity_per_grid, grid_values)
print(f"Realized Profit in USDC: {realized_profit}")

# Calculate final position assets with debug
final_ETH, final_USDC = calculate_end_position_assets_debug(price_sequence, initial_ETH, initial_USDC, quantity_per_grid, grid_values, initial_price)
print(f"Final ETH: {final_ETH}, Final USDC: {final_USDC}")

# Calculate final ETH price with debug
final_ETH_price_in_USDC = calculate_end_position_price_debug(price_sequence, grid_values, initial_price)
print(f"Final ETH Price in USDC: {final_ETH_price_in_USDC}")

# Calculate total values with debug
total_value_initial_USDC, total_value_final_USDC = calculate_total_values_debug(initial_ETH, initial_USDC, final_ETH, final_USDC, initial_price, final_ETH_price_in_USDC)
print(f"Total Initial Value in USDC: {total_value_initial_USDC}")
print(f"Total Final Value in USDC: {total_value_final_USDC}")

# Calculate final result in USDC
final_result_in_USDC = total_value_final_USDC - total_value_initial_USDC
print(f"Final Result in USDC: {final_result_in_USDC}")