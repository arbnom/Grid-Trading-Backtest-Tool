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

def create_price_sequence(data, grid_values):
    initial_price = data.iloc[0]['average_low_high']
    waiting_point = min(grid_values, key=lambda x: abs(x - initial_price))
    waiting_point_index = grid_values.index(waiting_point)

    sequence = []
    out_of_range = False

    for index, row in data.iterrows():
        price = row['average_low_high']

        if out_of_range and min(grid_values) <= price <= max(grid_values):
            out_of_range = False

        if not out_of_range:
            if waiting_point_index + 1 < len(grid_values) and price >= grid_values[waiting_point_index + 1]:
                sequence.append(1)
                waiting_point_index += 1

            elif waiting_point_index - 1 >= 0 and price <= grid_values[waiting_point_index - 1]:
                sequence.append(-1)
                waiting_point_index -= 1

        if price < min(grid_values) or price > max(grid_values):
            out_of_range = True

    return sequence

def calculate_required_assets(grid_values, initial_price, quantity_per_grid):
    waiting_point_index = grid_values.index(min(grid_values, key=lambda x: abs(x - initial_price)))
    required_ETH = sum(1 for i, val in enumerate(grid_values) if i > waiting_point_index) * quantity_per_grid
    required_USDC = sum(val for i, val in enumerate(grid_values) if i < waiting_point_index) * quantity_per_grid
    return required_ETH, required_USDC
    
def calculate_impermanent_loss(sequence, quantity_per_grid, grid_values, initial_ETH):
    diff_between_grids = grid_values[1] - grid_values[0]  # Assuming uniform grid spacing
    L = len(sequence)
    S = sum(sequence)
    impermanent_loss = 0  # Initialize to 0 to ensure it's always defined

    if S < 0:
        impermanent_loss = (quantity_per_grid * diff_between_grids * ((abs(S) - 1) * abs(S)) / 2)
    elif S > 0:
        impermanent_loss = (quantity_per_grid * diff_between_grids * ((S - 1) * S) / 2)

    print(f"Sequence: {sequence}, L: {L}, S: {S}, Theoretical Impermanent Loss: {impermanent_loss}")
    return impermanent_loss

def calculate_realized_profit(sequence, quantity_per_grid, grid_values):
    diff_between_grids = grid_values[1] - grid_values[0]  # Assuming uniform grid spacing
    L = len(sequence)
    S = sum(sequence)

    realized_profit = quantity_per_grid * diff_between_grids * (L - abs(S)) / 2
    return realized_profit

def calculate_end_position_assets(sequence, initial_ETH, initial_USDC, quantity_per_grid, grid_values, initial_price):
    waiting_point_index = grid_values.index(min(grid_values, key=lambda x: abs(x - initial_price)))
    final_ETH = initial_ETH
    final_USDC = initial_USDC

    for action in sequence:
        if action == 1:  # Sell order executed
            if waiting_point_index + 1 < len(grid_values):
                final_ETH -= quantity_per_grid
                final_USDC += quantity_per_grid * grid_values[waiting_point_index + 1]
                waiting_point_index += 1
        elif action == -1:  # Buy order executed
            if waiting_point_index - 1 >= 0:
                final_ETH += quantity_per_grid
                final_USDC -= quantity_per_grid * grid_values[waiting_point_index - 1]
                waiting_point_index -= 1

    return final_ETH, final_USDC

    waiting_point_index = grid_values.index(min(grid_values, key=lambda x: abs(x - initial_price)))

    for action in sequence:
        if action == 1 and waiting_point_index + 1 < len(grid_values):
            waiting_point_index += 1
        elif action == -1 and waiting_point_index - 1 >= 0:
            waiting_point_index -= 1

    # After applying all actions in the sequence, return the final ETH price
    final_ETH_price = grid_values[waiting_point_index]
    return final_ETH_price

def calculate_total_values(initial_ETH, initial_USDC, final_ETH, final_USDC, initial_ETH_price_in_USDC, final_ETH_price_in_USDC):
    total_value_initial_USDC = (initial_ETH * initial_ETH_price_in_USDC) + initial_USDC
    total_value_final_USDC = (final_ETH * final_ETH_price_in_USDC) + final_USDC
    return total_value_initial_USDC, total_value_final_USDC

# Parameters
csv_path = "path_to_the_csv_file.csv"
start_time = '2024-01-22T00:00:00Z'
end_time = '2024-01-23T00:00:00Z'
grid_range = (2400, 2500)
num_grids = 10
quantity_per_grid = 1

# Execution
grid_values = calculate_grid_values(csv_path, start_time, end_time, grid_range, num_grids)
filtered_data = pd.read_csv(csv_path)
filtered_data['time'] = pd.to_datetime(filtered_data['time'])
filtered_data = filtered_data[(filtered_data['time'] >= start_time) & (filtered_data['time'] <= end_time)]

# Get the initial price from the data
initial_price = filtered_data.iloc[0]['average_low_high']
print(f"Initial ETH Price in USDC: {initial_price}")

# Calculate required assets in terms of ETH and USDC
initial_ETH, initial_USDC = calculate_required_assets(grid_values, initial_price, quantity_per_grid)

# Calculate the total value of initial assets in terms of USDC
total_value_initial_USDC = (initial_ETH * initial_price) + initial_USDC
print(f"Required ETH: {initial_ETH}, Required USDC: {initial_USDC}")
print(f"Total Initial Value in USDC: {total_value_initial_USDC}")

# Generate the -1,1 sequence, pausing and resuming based on the price's position relative to the grid range
price_sequence = create_price_sequence(filtered_data, grid_values)

# Calculate the impermanent loss of the sequence
result = calculate_impermanent_loss(price_sequence, quantity_per_grid, grid_values, initial_ETH)

# Calculate the realized profit of the sequence
realized_profit = calculate_realized_profit(price_sequence, quantity_per_grid, grid_values)
print(f"Realized Profit in USDC: {realized_profit}")

# Get the final ETH price in USDC based on the sequence and grid values
final_ETH_price_in_USDC = filtered_data[filtered_data['time'] == end_time]['average_low_high'].iloc[-1]
print(f"Final ETH Price in USDC: {final_ETH_price_in_USDC}")

# Calculate final position assets at the end time
final_ETH, final_USDC = calculate_end_position_assets(price_sequence, initial_ETH, initial_USDC, quantity_per_grid, grid_values, initial_price)
print(f"Final ETH: {final_ETH}, Final USDC: {final_USDC}")

# Calculate the total value of final assets in terms of USDC
total_value_final_USDC = (final_ETH * final_ETH_price_in_USDC) + final_USDC
print(f"Total Final Value in USDC: {total_value_final_USDC}")

# Print the total result
print(f"Final Result in USDC: {total_value_final_USDC - total_value_initial_USDC}")