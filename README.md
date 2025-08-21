import pandas as pd
import numpy as np
from faker import Faker
import random

fake = Faker()

# Test dataset: 50 records
total_records = 50
unique_patients = 45   # 45 unique patients
revisit_patients = 5   # 5 patients revisit

# Gender distribution
gender_probs = [0.55, 0.45]  # Male 55%, Female 45%

# Countries, states, cities
countries = ["India", "USA", "Europe"]
states_per_country = 3
cities_per_state = 2

country_state_city = {}
for country in countries:
    states = [f"{country}_State_{i}" for i in range(1, states_per_country + 1)]
    country_state_city[country] = {}
    for state in states:
        cities = [f"{state}_City_{j}" for j in range(1, cities_per_state + 1)]
        country_state_city[country][state] = cities

# Generate unique patient IDs
patient_ids = np.arange(1, unique_patients + 1)
revisit_ids = np.random.choice(patient_ids, size=revisit_patients, replace=False)

records = []
patient_data = {}
patient_insurance_map = {}
patient_hospital_map = {}
patient_doctor_map = {}

for pid in patient_ids:
    country = random.choice(countries)
    state = random.choice(list(country_state_city[country].keys()))
    city = random.choice(country_state_city[country][state])
    
    gender = np.random.choice(["Male", "Female"], p=gender_probs)
    fullname = fake.name_male() if gender=="Male" else fake.name_female()
    email = f"{fullname.lower().replace(' ','.')}.{pid}@example.com"
    
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
        "patient_email": email,
        "patient_satisfactionscore": random.randint(1, 10),
        "patient_bloodgroup": random.choice(["A+", "A-", "B+", "B-", "AB+", "AB-", "O+", "O-"]),
        "patient_waittime": random.randint(5, 180)
    }
    
    patient_insurance_map[pid] = random.randint(1, 10)
    patient_hospital_map[pid] = random.randint(1, 10)
    patient_doctor_map[pid] = random.randint(1, 10)

# Generate hospital, doctor, department, insurance data
hospital_data = {i: {"hospital_name": fake.company()} for i in range(1, 11)}
doctor_data = {i: {"DoctorFullName": fake.name()} for i in range(1, 11)}
department_data = {i: {"departmentname": fake.bs()} for i in range(1, 6)}
insurance_data = {i: {"insuranceprovidername": fake.company()} for i in range(1, 11)}

visit_id = 1
all_patient_ids = list(patient_ids) + list(revisit_ids)

for pid in all_patient_ids:
    hospital_id = patient_hospital_map[pid]
    doctor_id = patient_doctor_map[pid]
    department_id = random.randint(1, 5)
    insurance_id = patient_insurance_map[pid]
    
    previous_visits = [r for r in records if r["PatientID"] == pid]
    if previous_visits:
        last_discharge = previous_visits[-1]["Discharge_Date"]
        admission_date = last_discharge + pd.to_timedelta(random.randint(30, 180), unit='d')
    else:
        admission_date = fake.date_between(start_date='-2y', end_date='today')
    
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
        "total_billing_amount": round(random.uniform(500, 5000), 2),
        "insurancecoveredamount": round(random.uniform(100, 3000), 2),
        "patient_covered_amount": round(random.uniform(50, 2000), 2),
        "full_date": discharge_date,
        "diagnosisid": random.randint(1, 20),
        "diagnosiscode": f"D{random.randint(100,199)}",
        "diagnosisdiscription": fake.sentence(),
        "diagnosiscategory": random.choice(["Critical", "Moderate", "Mild"]),
        "insuranceproviderid": insurance_id,
        **insurance_data[insurance_id]
    }
    
    records.append(record)
    visit_id += 1

df = pd.DataFrame(records)
df.to_csv("hospital_visits_50_records.csv", index=False)
print("CSV file with 50 records generated successfully!")
