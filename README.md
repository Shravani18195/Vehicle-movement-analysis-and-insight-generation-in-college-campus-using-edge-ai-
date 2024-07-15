# Vehicle-movement-analysis-and-insight-generation-in-college-campus-using-edge-ai-
Complete description on a Vehicle Movement Analysis project with code and report
import pandas as pd
import matplotlib.pyplot as plt
import seaborn as sns

# Load the data from the Excel file
file_path = '/content/vehicle details.xlsx'
df = pd.read_excel(file_path)

# Processing and Sorting of data
df['Entry Time'] = pd.to_datetime(df['Entry Time'], format='%H:%M:%S')
df['Exit Time'] = pd.to_datetime(df['Exit Time'], format='%H:%M:%S')
df.sort_values(by='Entry Time', inplace=True)

# Total number of parking slots available
total_slots = 25

# Creating time index and data frame to store occupancy data
time_index = pd.date_range(start=df['Entry Time'].min(), end=df['Exit Time'].max(), freq='30T')
occupancy_data = pd.DataFrame(0, index=time_index, columns=range(1, total_slots + 1))

# Calculating occupancy and assigning slots
slot_assignments = {}

for _, row in df.iterrows():
    entry_time = row['Entry Time']
    exit_time = row['Exit Time']

    for slot in range(1, total_slots + 1):
        if slot not in slot_assignments:
            slot_assignments[slot] = []

        is_free = all(not (entry <= entry_time <= exit or entry <= exit_time <= exit) for entry, exit in slot_assignments[slot])

        if is_free:
            slot_assignments[slot].append((entry_time, exit_time))
            occupancy_data.loc[entry_time:exit_time, slot] += 1
            break

# Creating summary DataFrame for slot usage frequency
slot_usage_summary = occupancy_data.sum(axis=0).reset_index()
slot_usage_summary.columns = ['Slot Number', 'Total Occupied Time (minutes)']
slot_usage_summary['Total Occupied Time (hours)'] = slot_usage_summary['Total Occupied Time (minutes)'] / 60

# Plotting the histogram for slot usage
plt.figure(figsize=(12, 6))
sns.barplot(x='Slot Number', y='Total Occupied Time (hours)', data=slot_usage_summary, color='blue')
plt.title('Slot Usage Over Time')
plt.xlabel('Slot Number')
plt.ylabel('Total Occupied Time (hours)')
plt.grid(True)
plt.tight_layout()
plt.show()

# Calculating and plotting the occupancy data
occupancy = occupancy_data.sum(axis=1)
available_slots = total_slots - occupancy

plt.figure(figsize=(12, 6))
plt.plot(occupancy.index.strftime('%H:%M'), occupancy, label='Occupancy', color='blue')
plt.plot(occupancy.index.strftime('%H:%M'), available_slots, label='Available Slots', color='green')
plt.axhline(total_slots, color='red', linestyle='--', label='Total Slots')
plt.fill_between(occupancy.index.strftime('%H:%M'), 0, occupancy, color='blue', alpha=0.3)
plt.title('Parking Lot Occupancy Over Time')
plt.xlabel('Time')
plt.ylabel('Number of Slots')
plt.legend()
plt.grid(True)
plt.tight_layout()
plt.show()

# Saving the occupancy and slot usage summary data to Excel files
occupancy_data.to_excel('occupancy_data.xlsx', index=True)
slot_usage_summary.to_excel('slot_usage_summary.xlsx', index=False)

# Printing peak occupancy and maximum available slots
print(f"Maximum no. of slots available: {total_slots}")
print(f"Slots Available at Peak occupancy: {available_slots.min()}")

# Calculating the total occupancy and total available slot
total_occupancy = occupancy_data.sum(axis=1)
total_available_slots = total_slots - occupancy_data.sum(axis=1)

# Finding the time with maximum occupancy and minimum occupancy
peak_occupancy_time = total_occupancy.idxmax()
peak_empty_slots_time = total_available_slots.idxmax()

print("Time at which most slots are filled:", peak_occupancy_time)
print("Time at which most slots are empty:", peak_empty_slots_time)
