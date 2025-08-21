import pandas as pd
import numpy as np
from faker import Faker
import random

fake = Faker()

# Total records and revisit count
total_records = 700_000
unique_patients = 675_000
revisit_patients = 25_000

# Gender distribution
gender_probs = [0.55, 0.45]  # Male 55%, Female 45%

# Countries, states, cities
countries = ["India", "USA", "Europe"]
states_per_country = 8
cities_per_state = 8

country_state_city = {}
for country in countries:
    states = [f"{country}_State_{i}" for i in range(1, states_per_country + 1)]
    country_state_city[country] = {}
    for state in states:
        cities = [f"{state}_City_{j}" for j in range(1, cities_per_state + 1)]
        country_state_city[country][state] = cities

# Generate unique patient IDs
patient_ids = np.arange(1, unique_patients + 1)

# Randomly select patients who will revisit
revisit_ids = np.random.choice(patient_ids, size=revisit_patients, replace=False)

# Initialize lists
records = []

# Generate unique patient data
patient_data = {}
patient_insurance_map = {}
patient_hospital_map = {}  # First hospital per patient
patient_doctor_map = {}    # First doctor per patient
patient_email_map = {}     # Email per patient

for pid in patient_ids:
    # Assign country, state, city randomly
    country = random.choice(countries)
    state = random.choice(list(country_state_city[country].keys()))
    city = random.choice(country_state_city[country][state])
    
    gender = np.random.choice(["Male", "Female"], p=gender_probs)
    
    fullname = fake.name_male() if gender=="Male" else fake.name_female()
    
    # Generate a unique email tied to patient ID
    email_local = fullname.lower().replace(" ", ".")
    email = f"{email_local}.{pid}@example.com"
    patient_email_map[pid] = email
    
    patient_data[pid] = {
        "Patient_Fullname": fullname,
        "Patient_Gender": gender,
        "Patient_age": random.randint(1, 90),
        "patientdob": fake.date_of_birth(minimum_age=1, maximum_age=90),
        "patientaddress": fake.address().replace("\n", ", "),
        "patientcity": city,
        "patientstate": state,
        "patientcountry": country,
        "patientpostalcode": fake.zipcode(),
        "patient_email": email,  # Consistent email per patient
        "patient_satisfactionscore": random.randint(1, 10),
        "patient_bloodgroup": random.choice(["A+", "A-", "B+", "B-", "AB+", "AB-", "O+", "O-"]),
        "patient_waittime": random.randint(5, 180)
    }
    
    # Assign consistent insurance, hospital, doctor per patient
    patient_insurance_map[pid] = random.randint(1, 50)
    patient_hospital_map[pid] = random.randint(1, 100)
    patient_doctor_map[pid] = random.randint(1, 200)

# Generate hospital, doctor, department, insurance data
hospital_data = {i: {
    "hospital_name": fake.company(),
    "hospital_provideer_name": fake.company_suffix(),
    "hospital_type": random.choice(["Government", "Private"]),
    "bed_capacity": random.randint(50, 500),
    "hospital_address": fake.address().replace("\n", ", "),
    "hospitalstate": random.choice([state for country in countries for state in country_state_city[country].keys()]),
    "hospitalcity": fake.city(),
    "hospitalcountry": random.choice(countries),
    "hospitalpostalcode": fake.zipcode()
} for i in range(1, 101)}

doctor_data = {i: {
    "DoctorFullName": fake.name(),
    "doctor_specialization": random.choice(["Cardiology", "Neurology", "Orthopedics", "General", "Pediatrics"]),
    "doctoryears_experience": random.randint(1, 40),
    "doctoremail": fake.email(),
    "doctor_shift_type": random.choice(["Morning", "Evening", "Night"])
} for i in range(1, 201)}

department_data = {i: {
    "departmentname": fake.bs(),
    "department_speciality": random.choice(["Surgery", "Diagnostics", "Treatment"]),
    "Departmentreferral": random.choice([True, False]),
    "floor_number": random.randint(1, 10)
} for i in range(1, 51)}

insurance_data = {i: {
    "insuranceprovidername": fake.company(),
    "insuranceplan_type": random.choice(["Basic", "Premium", "Gold"]),
    "coverage_percentage": random.randint(50, 100),
    "insurance_email": fake.email()
} for i in range(1, 51)}

# Generate visit records
visit_id = 1
all_patient_ids = list(patient_ids) + list(revisit_ids)

for pid in all_patient_ids:
    # Use the same hospital and doctor if revisit
    hospital_id = patient_hospital_map[pid]
    doctor_id = patient_doctor_map[pid]
    department_id = random.randint(1, 50)
    insurance_id = patient_insurance_map[pid]
    
    # Check if this patient has visited before
    previous_visits = [r for r in records if r["PatientID"] == pid]
    if previous_visits:
        # Revisit: after last discharge date, add 30-180 days
        last_discharge = previous_visits[-1]["Discharge_Date"]
        admission_date = last_discharge + pd.to_timedelta(random.randint(30, 180), unit='d')
    else:
        # First visit: random date in past 5 years
        admission_date = fake.date_between(start_date='-5y', end_date='today')
    
    discharge_date = admission_date + pd.to_timedelta(random.randint(1, 15), unit='d')
    
    record = {
        "VisitID": visit_id,
        "Admission_Date": admission_date,
        "Discharge_Date": discharge_date,
        "PatientID": pid,
        **patient_data[pid],
        "DoctorID": doctor_id,
        **doctor_data[doctor_id],
        "departmentid": department_id,
        **department_data[department_id],
        "hospitalid": hospital_id,
        **hospital_data[hospital_id],
        "total_billing_amount": round(random.uniform(500, 50000), 2),
        "insurancecoveredamount": round(random.uniform(100, 30000), 2),
        "patient_covered_amount": round(random.uniform(50, 20000), 2),
        "full_date": discharge_date,
        "diagnosisid": random.randint(1, 1000),
        "diagnosiscode": f"D{random.randint(100,999)}",
        "diagnosisdiscription": fake.sentence(),
        "diagnosiscategory": random.choice(["Critical", "Moderate", "Mild"]),
        "insuranceproviderid": insurance_id,
        **insurance_data[insurance_id]
    }
    
    records.append(record)
    visit_id += 1

# Create DataFrame and save CSV
df = pd.DataFrame(records)
df.to_csv("hospital_visits_700k_email_matched.csv", index=False)
print("CSV file generated successfully with 7 lakh records and emails matching patient IDs!")
