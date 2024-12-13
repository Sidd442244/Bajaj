import pandas as pd
import json

# Load the JSON data
with open('DataEngineeringQ2.json', 'r') as f:
    data = json.load(f)

# Initialize an empty list to store each flattened record
flattened_data = []

# Iterate over each entry in the JSON array
for entry in data:
    # Extract patient details and consultation data (medicines)
    patient_details = entry['patientDetails']
    consultation_data = entry['consultationData']
    medicines = consultation_data['medicines']

    # Flatten the patient details
    patient_df = pd.json_normalize(patient_details)

    # Flatten medicines list inside consultation data
    medicines_df = pd.json_normalize(medicines)

    # Combine patient details and medicines (you can choose how to join)
    combined_df = pd.concat([patient_df, medicines_df], axis=1)

    # Append the combined data for this entry
    flattened_data.append(combined_df)

# Combine all the flattened entries into one DataFrame
final_df = pd.concat(flattened_data, ignore_index=True)

# Display the columns and first few rows to inspect the structure
print(final_df.columns)
print(final_df.head())

from datetime import datetime

# Convert birthDate to age, adjusting the format to handle the timestamp
final_df['age'] = final_df['birthDate'].apply(
    lambda x: (datetime.now() - datetime.strptime(str(x), '%Y-%m-%dT%H:%M:%S.%fZ')).days // 365 if pd.notnull(x) else None
)
print(final_df)
# Count the number of adults (ages 20 to 59)
adult_count = final_df[(final_df['age'] >= 20) & (final_df['age'] <= 59)].shape[0]

print(adult_count)

# Calculate the average number of medicines prescribed per patient
average_medicines = final_df['medicineName'].notna().groupby(final_df.index).sum().mean()
print(round(average_medicines, 2))
# Find the 3rd most frequently prescribed medicine
medicine_counts = final_df['medicineName'].value_counts()
third_most_frequent_medicine = medicine_counts.index[2] if len(medicine_counts) >= 3 else None
print(third_most_frequent_medicine)
# Calculate the percentage of missing values for specified columns
missing_percentage = final_df[['firstName', 'lastName', 'birthDate']].isnull().mean() * 100
print(f"{round(missing_percentage['firstName'], 2)}, {round(missing_percentage['lastName'], 2)}, {round(missing_percentage['birthDate'], 2)}")
# Impute missing gender values with the mode (most frequent value)
mode_gender = final_df['gender'].mode()[0]
final_df['gender'].fillna(mode_gender, inplace=True)

# Calculate the percentage of females
female_percentage = (final_df['gender'] == 'Female').mean() * 100
print(round(female_percentage, 2))
# Calculate the percentage distribution of active vs inactive medicines
active_percentage = (final_df['isActive'] == True).mean() * 100
inactive_percentage = 100 - active_percentage
print(f"{round(active_percentage, 2)}, {round(inactive_percentage, 2)}")

#question 2 3 4
# Convert the 'duration' column to numeric, forcing any errors to NaN
final_df['duration'] = pd.to_numeric(final_df['duration'], errors='coerce')

# Calculate the average duration, ignoring NaN values
average_duration = final_df['duration'].mean()

# Round the result to 2 decimal places
print(round(average_duration, 2))

# Check for missing values across all entries
missing_values = final_df.isnull().sum()

print("Missing values in the dataset:")
print(missing_values)
# Analyze the most common medicine names across all entries
most_common_medicine = final_df['medicineName'].value_counts().head(1)

# Analyze the most common frequency of medicine prescriptions
most_common_frequency = final_df['frequency'].value_counts().head(1)

print("Most common medicine prescribed:")
print(most_common_medicine)

print("Most common frequency of prescription:")
print(most_common_frequency)


import pandas as pd
import json

# Load the JSON data (replace with actual file path)
with open('DataEngineeringQ2.json', 'r') as file:
    data = json.load(file)

# Initialize an empty list to hold the extracted phone numbers
phone_numbers = []

# Loop through each entry in the JSON data and extract 'phoneNumber'
for entry in data:
    # Extract the 'phoneNumber' if it exists in the entry
    phone_number = entry.get('phoneNumber', None)
    phone_numbers.append(phone_number)

# Create a DataFrame from the extracted phone numbers
final_df = pd.DataFrame(phone_numbers, columns=['phoneNumber'])

# Define the function to validate the phone numbers
def is_valid_phone_number(phone_number):
    # Remove non-numeric characters
    phone_number = ''.join(filter(str.isdigit, str(phone_number)))
    
    # Check if the number is valid according to the rules
    if len(phone_number) == 10 and phone_number[0] in ['6', '7', '8', '9']:
        return True
    elif phone_number.startswith('+91') and len(phone_number) == 13:
        return True
    elif phone_number.startswith('91') and len(phone_number) == 12:
        return True
    return False

# Apply the validation function to the 'phoneNumber' column and create 'isValidMobile' column
final_df['isValidMobile'] = final_df['phoneNumber'].apply(is_valid_phone_number)

# Count the number of valid phone numbers
valid_phone_count = final_df['isValidMobile'].sum()
print(f"Number of valid phone numbers: {valid_phone_count}")

# Display the DataFrame with the new columns
print(final_df.head())


import pandas as pd
import json
from datetime import datetime

# Load the JSON data (replace with actual file path)
with open('DataEngineeringQ2.json', 'r') as file:
    data = json.load(file)

# Initialize lists to hold extracted data
ages = []
num_medicines = []

# Loop through each entry in the JSON data to extract 'birthDate' and 'medicines'
for entry in data:
    # Calculate the age based on 'birthDate'
    birth_date = entry.get('patientDetails', {}).get('birthDate', None)
    if birth_date:
        birth_date = datetime.strptime(birth_date.split('T')[0], '%Y-%m-%d')
        age = (datetime.now() - birth_date).days // 365
    else:
        age = None

    # Count the number of prescribed medicines (length of 'medicines' list)
    medicines = entry.get('consultationData', {}).get('medicines', [])
    num_medicines.append(len(medicines))
    ages.append(age)

# Create a DataFrame
df = pd.DataFrame({
    'age': ages,
    'num_medicines': num_medicines
})

# Drop rows where age or number of medicines is None
df.dropna(subset=['age', 'num_medicines'], inplace=True)

# Calculate Pearson correlation
correlation = df['age'].corr(df['num_medicines'])

print(f"Pearson correlation between age and number of prescribed medicines: {correlation}")
