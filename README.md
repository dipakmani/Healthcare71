import pandas as pd
import numpy as np
from faker import Faker
import random

# Initialize Faker
fake = Faker()

# Number of records
num_records = 500000

# Countries
countries = ['India', 'USA', 'UK', 'Australia']

# Genders with male 10% more than female
genders = ['Male', 'Female']
gender_weights = [0.55, 0.45]

# Blood groups
blood_groups = ['A+', 'A-', 'B+', 'B-', 'AB+', 'AB-', 'O+', 'O-']

# Doctor specializations
specializations = ['Cardiology', 'Neurology', 'Orthopedics', 'Oncology', 'Pediatrics', 'General']

# Department info
departments = ['Cardiology', 'Neurology', 'Orthopedics', 'Oncology', 'Pediatrics', 'General Medicine']
department_speciality = ['Heart', 'Brain', 'Bones', 'Cancer', 'Child', 'General']

# Hospital types
hospital_types = ['Government', 'Private', 'Multi-speciality', 'Clinic']

# Insurance plans
insurance_plans = ['Gold', 'Silver', 'Platinum', 'Basic']

# Helper to generate random dates
def random_date(start_year=2015, end_year=2025):
    return fake.date_between(start_date=f'{start_year}-01-01', end_date=f'{end_year}-12-31')

# Generate dataset
data = []

for i in range(1, num_records + 1):
    admission_date = random_date()
    discharge_date = admission_date + pd.Timedelta(days=random.randint(1, 20))
    patient_dob = fake.date_of_birth(minimum_age=1, maximum_age=90)
    patient_age = (admission_date - patient_dob).days // 365
    patient_country = random.choice(countries)
    
    total_billing = round(random.uniform(1000, 10000), 2)
    insurance_covered = round(total_billing * random.uniform(0.5, 0.9), 2)
    patient_covered = round(total_billing - insurance_covered, 2)
    
    record = [
        i,  # VisitID
        admission_date,
        discharge_date,
        i,  # PatientID
        fake.name(),
        random.choices(genders, weights=gender_weights, k=1)[0],
        patient_age,
        patient_dob,
        fake.address().replace("\n", ", "),
        fake.city(),
        fake.state(),
        patient_country,
        fake.postcode(),
        round(random.uniform(1, 5), 1),  # patient_satisfactionscore
        random.choice(blood_groups),
        random.randint(5, 120),  # patient_waittime
        i,  # DoctorID
        fake.name(),
        random.randint(1, 40),  # doctoryears_experience
        random.choice(specializations),
        fake.email(),
        random.choice(['Morning', 'Evening', 'Night']),
        random.randint(1, 10),  # departmentid
        random.choice(departments),
        random.choice(department_speciality),
        random.choice(['Yes', 'No']),
        random.randint(1, 50),  # hospitalid
        fake.company(),
        random.choice(hospital_types),
        random.randint(50, 500),  # bed_capacity
        fake.address().replace("\n", ", "),
        fake.state(),
        fake.city(),
        patient_country,
        fake.postcode(),
        total_billing,
        insurance_covered,
        patient_covered,
        fake.date_this_decade(),
        random.randint(1, 20),  # insuranceproviderid
        fake.company(),
        random.choice(insurance_plans),
        round(random.uniform(50, 90), 2),  # coverage_percentage
        fake.email()
    ]
    
    data.append(record)

# Columns
columns = [
    'VisitID', 'Admission_Date', 'Discharge_Date', 'PatientID', 'Patient_Fullname', 'Patient_Gender', 'Patient_age',
    'patientdob', 'patientaddress', 'patientcity', 'patientstate', 'patientcountry', 'patientpostalcode',
    'patient_satisfactionscore', 'patient_bloodgroup', 'patient_waittime', 'DoctorID', 'DoctorFullName',
    'doctoryears_experience', 'doctor_specialization', 'doctoremail', 'doctor_shift_type', 'departmentid',
    'departmentname', 'department_speciality', 'Departmentreferral', 'hospitalid', 'hospital_provideer_name',
    'hospital_type', 'bed_capacity', 'hospital_address', 'hospitalstate', 'hospitalcity', 'hospitalcountry',
    'hospitalpostalcode', 'total_billing_amount', 'insurancecoveredamount', 'patient_covered_amount', 'full_date',
    'insuranceproviderid', 'insuranceprovidername', 'insuranceplan_type', 'coverage_percentage', 'insurance_email'
]

# Create DataFrame
df = pd.DataFrame(data, columns=columns)

# Save to CSV
df.to_csv('synthetic_healthcare_data.csv', index=False)
print("500,000 records generated successfully as 'synthetic_healthcare_data.csv'")
