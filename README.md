# PYTHON-TASK-2
________________________

Question 1: Calculate Distance Matrix


import pandas as pd

def calculate_distance_matrix(dataset):
    
    distance_matrix = dataset.pivot_table(index='id_start', columns='id_end', values='distance', aggfunc='sum', fill_value=0)

    distance_matrix = distance_matrix + distance_matrix.T - pd.DataFrame(np.diag(distance_matrix.values), index=distance_matrix.index, columns=distance_matrix.columns)

    return distance_matrix

# Example usage
dataset_path = 'dataset-3.csv'
df = pd.read_csv(dataset_path)
result_distance_matrix = calculate_distance_matrix(df)

# Print the resulting distance matrix
print(result_distance_matrix)

-------------------------------------------------
Question 2: Unroll Distance Matrix

def unroll_distance_matrix(distance_matrix):
    distances_df = distance_matrix.stack().reset_index()
    
    distances_df.columns = ['id_start', 'id_end', 'distance']

    distances_df = distances_df[distances_df['id_start'] != distances_df['id_end']]

    return distances_df

# Example usage
result_unrolled_distances = unroll_distance_matrix(result_distance_matrix)

# Print the resulting unrolled distance DataFrame
print(result_unrolled_distances)

--------------------------------------------------------------------
Question 3: Find IDs within Ten Percentage Threshold

def find_ids_within_ten_percentage_threshold(unrolled_distances, reference_value):
    avg_distance = unrolled_distances[unrolled_distances['id_start'] == reference_value]['distance'].mean()

    threshold_min = avg_distance * 0.9
    threshold_max = avg_distance * 1.1

    result_ids = unrolled_distances[(unrolled_distances['distance'] >= threshold_min) & (unrolled_distances['distance'] <= threshold_max)]['id_start'].unique()
    result_ids.sort()

    return result_ids

# Example usage
reference_value = 1  # Replace with the desired reference value
result_within_threshold = find_ids_within_ten_percentage_threshold(result_unrolled_distances, reference_value)

# Print the resulting list of ids within ten percentage threshold
print(result_within_threshold)

----------------------------------------------------------
Question 4: Calculate Toll Rate


def calculate_toll_rate(unrolled_distances):
    toll_rates_df = unrolled_distances.copy()

    rate_coefficients = {'moto': 0.8, 'car': 1.2, 'rv': 1.5, 'bus': 2.2, 'truck': 3.6}

    for vehicle_type, rate_coefficient in rate_coefficients.items():
        toll_rates_df[vehicle_type] = toll_rates_df['distance'] * rate_coefficient

    return toll_rates_df

# Example usage
result_toll_rates = calculate_toll_rate(result_unrolled_distances)

# Print the resulting toll rates DataFrame
print(result_toll_rates)

---------------------------------------------------------------
Question 5: Calculate Time-Based Toll Rates


import datetime

def calculate_time_based_toll_rates(unrolled_distances):
    time_based_toll_rates_df = unrolled_distances.copy()

    weekday_time_ranges = [(datetime.time(0, 0, 0), datetime.time(10, 0, 0)),
                           (datetime.time(10, 0, 0), datetime.time(18, 0, 0)),
                           (datetime.time(18, 0, 0), datetime.time(23, 59, 59))]

    weekend_time_range = (datetime.time(0, 0, 0), datetime.time(23, 59, 59))

    time_based_toll_rates_df['discount_factor'] = 1.0  # Initialize with no discount

    for start_time, end_time in weekday_time_ranges:
        time_based_toll_rates_df.loc[
            (time_based_toll_rates_df['start_time'] >= start_time) & (time_based_toll_rates_df['end_time'] <= end_time) &
            (time_based_toll_rates_df['start_day'].isin(['Monday', 'Tuesday', 'Wednesday', 'Thursday', 'Friday'])),
            'discount_factor'
        ] = 0.8

    time_based_toll_rates_df.loc[
        time_based_toll_rates_df['start_day'].isin(['Saturday', 'Sunday']),
        'discount_factor'
    ] = 0.7

    time_based_toll_rates_df['time_based_toll'] = time_based_toll_rates_df['distance'] * time_based_toll_rates_df['discount_factor']

    return time_based_toll_rates_df

# Example usage
result_time_based_toll_rates = calculate_time_based_toll_rates(result_unrolled_distances)

# Print the resulting time-based toll rates DataFrame
print(result_time_based_toll_rates)
