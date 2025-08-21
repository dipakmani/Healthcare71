import pandas as pd
import numpy as np
from faker import Faker
import random

fake = Faker()

# -----------------------
# PARAMETERS
# -----------------------
total_patients = 475000    # unique patients
revisit_patients = 25000   # patients who revisit
total_records = 500000     # total visits

# -----------------------
# GENERATE UNIQUE PATIENTS WITH GENDER BIAS
# -----------------------
patients = []
for i in range(1, total_patients + 1):
    gender = np.random.choice(['Male', 'Female'], p=[0.55, 0.45])  # Male slightly higher
    dob = fake.date_of_birth(minimum_age=0, maximum_age=90)
    patient = {
        'PatientID': i,
        'Patient_Fullname': fake.name_male() if gender=='Male' else fake.name_female(),
        'Patient_Gender': gender,
        'Patient_age': (pd.Timestamp('today') - pd.Timestamp(dob)).days // 365,
        'patientdob': dob,
        'patientaddress': fake.address().replace('\n', ', '),
        'patientcity': fake.city(),
        'patientstate': fake.state(),
        'patientcountry': fake.country(),
        'patientpostalcode': fake.postcode(),
        'patient_satisfactionscore': random.randint(1, 10),
        'patient_bloodgroup': random.choice(['A+', 'A-', 'B+', 'B-', 'AB+', 'AB-', 'O+', 'O-']),
        'patient_waittime': random.randint(5, 180),
        'patient_email': fake.email()
    }
    patients.append(patient)

patients_df = pd.DataFrame(patients)

# -----------------------
# GENERATE REVISITS
# -----------------------
revisit_ids = np.random.choice(patients_df['PatientID'], size=revisit_patients, replace=False)
revisits_df = patients_df[patients_df['PatientID'].isin(revisit_ids)].copy()

# -----------------------
# COMBINE UNIQUE + REVISITS
# -----------------------
visits_df = pd.concat([patients_df, revisits_df], ignore_index=True)

# Ensure exact total_records
if len(visits_df) < total_records:
    extra_rows = total_records - len(visits_df)
    extra_patients = patients_df.sample(extra_rows, replace=True)
    visits_df = pd.concat([visits_df, extra_patients], ignore_index=True)

# Shuffle visits
visits_df = visits_df.sample(frac=1).reset_index(drop=True)

# -----------------------
# VISITID AND DATES
# -----------------------
visits_df['VisitID'] = range(1, len(visits_df)+1)

# Admission_Date: random date within last 3 years
visits_df['Admission_Date'] = pd.to_datetime(
    [fake.date_between(start_date='-3y', end_date='today') for _ in range(len(visits_df))]
)

# Discharge_Date: Admission_Date + 1-14 days
visits_df['Discharge_Date'] = visits_df['Admission_Date'] + pd.to_timedelta(
    np.random.randint(1, 15, size=len(visits_df)), unit='D'
)

# -----------------------
# DOCTORS
# -----------------------
doctors = []
for i in range(1, 2001):
    doctors.append({
        'DoctorID': i,
        'DoctorFullName': fake.name(),
        'doctor_specialization': random.choice(['Cardiology', 'Neurology', 'Orthopedics', 'Pediatrics', 'General']),
        'doctoryears_experience': random.randint(1, 40),
        'doctoremail': fake.email(),
        'doctor_shift_type': random.choice(['Morning', 'Evening', 'Night'])
    })
doctors_df = pd.DataFrame(doctors)

# -----------------------
# DEPARTMENTS
# -----------------------
departments = []
for i in range(1, 101):
    departments.append({
        'departmentid': i,
        'departmentname': fake.word().title() + " Dept",
        'department_speciality': random.choice(['Surgery', 'Medicine', 'Radiology', 'Cardiology']),
        'Departmentreferral': random.choice(['Yes', 'No']),
        'floor_number': random.randint(1, 10)
    })
departments_df = pd.DataFrame(departments)

# -----------------------
# HOSPITALS
# -----------------------
hospitals = []
for i in range(1, 51):
    hospitals.append({
        'hospitalid': i,
        'hospital_name': fake.company() + " Hospital",
        'hospital_provideer_name': fake.company(),
        'hospital_type': random.choice(['Private', 'Public']),
        'bed_capacity': random.randint(50, 500),
        'hospital_address': fake.address().replace('\n', ', '),
        'hospitalstate': fake.state(),
        'hospitalcity': fake.city(),
        'hospitalcountry': fake.country(),
        'hospitalpostalcode': fake.postcode()
    })
hospitals_df = pd.DataFrame(hospitals)

# -----------------------
# DIAGNOSIS
# -----------------------
diagnosis_categories = ['Bones', 'Fever', 'Skin', 'Cancer', 'Cardiology', 'Neurology', 'Pediatrics', 'Respiratory', 'Diabetes', 'General']
diagnosis = []
for i in range(1, 1001):
    diagnosis.append({
        'diagnosisid': i,
        'diagnosiscode': f"D{i:04d}",
        'diagnosisdiscription': fake.sentence(nb_words=6),
        'diagnosiscategory': random.choice(diagnosis_categories)
    })
diagnosis_df = pd.DataFrame(diagnosis)

# -----------------------
# INSURANCE
# -----------------------
insurance = []
for i in range(1, 201):
    insurance.append({
        'insuranceproviderid': i,
        'insuranceprovidername': fake.company(),
        'insuranceplan_type': random.choice(['Basic', 'Premium', 'Gold', 'Platinum']),
        'coverage_percentage': random.randint(50, 100),
        'insurance_email': fake.email()
    })
insurance_df = pd.DataFrame(insurance)

# -----------------------
# MERGE DOCTOR, DEPT, HOSPITAL, DIAGNOSIS, INSURANCE
# -----------------------
visits_df['DoctorID'] = np.random.choice(doctors_df['DoctorID'], size=len(visits_df))
visits_df = visits_df.merge(doctors_df, on='DoctorID', how='left')

visits_df['departmentid'] = np.random.choice(departments_df['departmentid'], size=len(visits_df))
visits_df = visits_df.merge(departments_df, on='departmentid', how='left')

visits_df['hospitalid'] = np.random.choice(hospitals_df['hospitalid'], size=len(visits_df))
visits_df = visits_df.merge(hospitals_df, on='hospitalid', how='left')

visits_df['diagnosisid'] = np.random.choice(diagnosis_df['diagnosisid'], size=len(visits_df))
visits_df = visits_df.merge(diagnosis_df, on='diagnosisid', how='left')

visits_df['insuranceproviderid'] = np.random.choice(insurance_df['insuranceproviderid'], size=len(visits_df))
visits_df = visits_df.merge(insurance_df, on='insuranceproviderid', how='left')

# -----------------------
# BILLING
# -----------------------
visits_df['total_billing_amount'] = np.random.randint(100, 2000, size=len(visits_df))
visits_df['insurancecoveredamount'] = (visits_df['total_billing_amount'] * visits_df['coverage_percentage'] / 100).astype(int)
visits_df['patient_covered_amount'] = visits_df['total_billing_amount'] - visits_df['insurancecoveredamount']

# -----------------------
# FULL DATE COLUMN
# -----------------------
visits_df['full_date'] = visits_df['Admission_Date']

# -----------------------
# SAVE CSV
# -----------------------
visits_df.to_csv('healthcare_dataset_5lakh.csv', index=False)
print("CSV with 5 lakh records including patient emails, gender bias, and proper admission/discharge dates generated successfully!")
