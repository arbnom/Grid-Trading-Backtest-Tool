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
            out_of_range = False  # Price re-entered the range

        if not out_of_range:
            if waiting_point_index + 1 < len(grid_values) and price >= grid_values[waiting_point_index + 1]:
                sequence.append(1)
                waiting_point_index += 1
            elif waiting_point_index - 1 >= 0 and price <= grid_values[waiting_point_index - 1]:
                sequence.append(-1)
                waiting_point_index -= 1

        if price < min(grid_values) or price > max(grid_values):
            out_of_range = True  # Price went outside of the range

    return sequence

def calculate_required_assets(grid_values, initial_price, quantity_per_grid):
    waiting_point_index = grid_values.index(min(grid_values, key=lambda x: abs(x - initial_price)))
    required_ETH = sum(1 for i, val in enumerate(grid_values) if i > waiting_point_index) * quantity_per_grid
    required_USDC = sum(val for i, val in enumerate(grid_values) if i < waiting_point_index) * quantity_per_grid
    return required_ETH, required_USDC

# Parameters
csv_path = 'path_to_your_csv_file.csv'  # Update the path to your CSV file
start_time = '2024-01-22T00:00:00Z'
end_time = '2024-01-23T00:00:00Z'
grid_range = (2400, 2500)  # Example range (low, high)
num_grids = 10
quantity_per_grid = 0.1  # Example quantity per grid

# Execution
grid_values = calculate_grid_values(csv_path, start_time, end_time, grid_range, num_grids)
filtered_data = pd.read_csv(csv_path)
filtered_data['time'] = pd.to_datetime(filtered_data['time'])
filtered_data = filtered_data[(filtered_data['time'] >= start_time) & (filtered_data['time'] <= end_time)]

initial_price = filtered_data.iloc[0]['average_low_high']
required_ETH, required_USDC = calculate_required_assets(grid_values, initial_price, quantity_per_grid)
print(f"Required ETH: {required_ETH}, Required USDC: {required_USDC}")

price_sequence = create_price_sequence(filtered_data, grid_values)
print(price_sequence)
