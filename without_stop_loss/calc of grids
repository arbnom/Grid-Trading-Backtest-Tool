import pandas as pd

def calculate_grid_values(start_time, end_time, grid_range, num_grids, quantity_per_grid):
    # Load the CSV file
    df = pd.read_csv('averaged_low_high.csv')
    
    # Convert the 'time' column to datetime
    df['time'] = pd.to_datetime(df['time'])
    
    # Filter the DataFrame based on the time range
    mask = (df['time'] >= start_time) & (df['time'] <= end_time)
    df_filtered = df.loc[mask]
    
    # Ensure the DataFrame is not empty after filtering
    if df_filtered.empty:
        raise ValueError("No data available in the specified time range.")
    
    # Calculate the grid values
    low, high = grid_range
    grid_size = (high - low) / (num_grids)
    grid_values = [low + i * grid_size for i in range(num_grids + 1)]
    
    return grid_values

# Example usage
start_time = '2024-01-22T00:00:00Z'
end_time = '2024-01-23T00:00:00Z'
grid_range = (2000, 2050)  # Example range (low, high)
num_grids = 5
quantity_per_grid = 0.1

grid_values = calculate_grid_values(start_time, end_time, grid_range, num_grids, quantity_per_grid)
print(grid_values)
