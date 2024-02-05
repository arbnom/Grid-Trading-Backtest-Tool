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

    for index, row in data.iterrows():
        price = row['average_low_high']

        # Update the sequence only when the price crosses to a new grid point
        if price >= grid_values[min(waiting_point_index + 1, len(grid_values) - 1)] and waiting_point != grid_values[min(waiting_point_index + 1, len(grid_values) - 1)]:
            sequence.append(1)
            waiting_point_index = min(waiting_point_index + 1, len(grid_values) - 1)
            waiting_point = grid_values[waiting_point_index]

        elif price <= grid_values[max(waiting_point_index - 1, 0)] and waiting_point != grid_values[max(waiting_point_index - 1, 0)]:
            sequence.append(-1)
            waiting_point_index = max(waiting_point_index - 1, 0)
            waiting_point = grid_values[waiting_point_index]

        # Stop sequence generation if the price goes outside of the grid range
        if price < min(grid_values) or price > max(grid_values):
            break

    return sequence

def calculate_required_assets(grid_values, initial_price, quantity_per_grid):
    grids_above = sum(i > initial_price for i in grid_values)
    grids_below = sum(i < initial_price for i in grid_values)

    required_ETH = grids_above * quantity_per_grid
    required_USDC = sum(grid_values[i] for i in range(len(grid_values)) if grid_values[i] < initial_price) * quantity_per_grid

    print(f"Required ETH: {required_ETH}, Required USDC: {required_USDC}")
    return required_ETH, required_USDC

# Parameters
csv_path = "path_to_the_csv_file.csv"
start_time = '2024-01-22T00:00:00Z'
end_time = '2024-01-23T00:00:00Z'
grid_range = (2000, 3000)
num_grids = 100
quantity_per_grid = 0.1

# Execute the functions
grid_values = calculate_grid_values(csv_path, start_time, end_time, grid_range, num_grids)

# Load the filtered data
filtered_data = pd.read_csv(csv_path)
filtered_data['time'] = pd.to_datetime(filtered_data['time'])
filtered_data = filtered_data[(filtered_data['time'] >= start_time) & (filtered_data['time'] <= end_time)]

# Generate the -1,1 sequence and stop if the price goes outside of the grid range
price_sequence = create_price_sequence(filtered_data, grid_values)
print(price_sequence)

# Calculate required ETH and USDC for the position
initial_price = filtered_data.iloc[0]['average_low_high']
calculate_required_assets(grid_values, initial_price, quantity_per_grid)
