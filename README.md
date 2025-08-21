import pandas as pd
import numpy as np
from faker import Faker
import random

# -----------------------------
# Initialize Faker and Random Seed
# -----------------------------
fake = Faker()
np.random.seed(42)
random.seed(42)

# -----------------------------
# Parameters
# -----------------------------
NUM_PATIENTS = 200000
NUM_VISITS = 500000
NUM_DOCTORS = 100
NUM_OPERATIONS_PATIENTS = 100000  # 50% patients
NUM_EQUIPMENT = 50
NUM_PROCEDURES = 30
NUM_LABTESTS = 50
NUM_MEDICATIONS = 100

# -----------------------------
# Generate Unique Patients
# -----------------------------
patient_ids = [f'Patient_{i:06d}' for i in range(1, NUM_PATIENTS+1)]
patient_data = []

for pid in patient_ids:
    patient_data.append({
        'PatientID': pid,
        'PatientName': fake.name(),
        'Gender': random.choice(['Male', 'Female']),
        'DOB': fake.date_of_birth(minimum_age=0, maximum_age=90),
        'City': fake.city(),
        'State': fake.state(),
        'Country': fake.country(),
        'BloodGroup': random.choice(['A+', 'A-', 'B+', 'B-', 'AB+', 'AB-', 'O+', 'O-'])
    })

df_patients = pd.DataFrame(patient_data)

# -----------------------------
# Generate Visits
# -----------------------------
visit_ids = [f'Visit_{i:06d}' for i in range(1, NUM_VISITS+1)]
visit_patient_ids = np.random.choice(patient_ids, size=NUM_VISITS, replace=True)

visit_data = []
for vid, pid in zip(visit_ids, visit_patient_ids):
    visit_data.append({
        'VisitID': vid,
        'PatientID': pid,
        'VisitDate': fake.date_between(start_date='-2y', end_date='today'),
        'DoctorID': f'Doctor_{random.randint(1, NUM_DOCTORS):03d}',
        'DoctorVisitFee': random.randint(100, 1500)
    })

df_visits = pd.DataFrame(visit_data)

# -----------------------------
# Assign Operations to 50% patients
# -----------------------------
operation_patient_ids = np.random.choice(patient_ids, size=NUM_OPERATIONS_PATIENTS, replace=False)
operation_data = []
operation_ids = [f'Operation_{i:06d}' for i in range(1, NUM_OPERATIONS_PATIENTS+1)]

for op_id, pid in zip(operation_ids, operation_patient_ids):
    visit_dates = df_visits[df_visits['PatientID'] == pid]['VisitDate'].sort_values().tolist()
    visit_date = visit_dates[-1]  # latest visit date for operation
    operation_date = visit_date + pd.Timedelta(days=random.randint(1,30))
    discharge_date = operation_date + pd.Timedelta(days=random.randint(1,10))
    
    operation_data.append({
        'OperationID': op_id,
        'PatientID': pid,
        'OperationDate': operation_date,
        'DischargeDate': discharge_date,
        'ProcedureID': f'Procedure_{random.randint(1, NUM_PROCEDURES):03d}',
        'RoomID': f'Room_{random.randint(1, 50):03d}',
        'EquipmentID': f'Equipment_{random.randint(1, NUM_EQUIPMENT):03d}'
    })

df_operations = pd.DataFrame(operation_data)

# -----------------------------
# Generate Lab Tests
# -----------------------------
lab_data = []
for pid in operation_patient_ids:
    num_tests = random.randint(1,5)
    for _ in range(num_tests):
        lab_data.append({
            'PatientID': pid,
            'LabTestID': f'LabTest_{random.randint(1, NUM_LABTESTS):03d}',
            'TestDate': fake.date_between(start_date='-2y', end_date='today')
        })

df_labtests = pd.DataFrame(lab_data)

# -----------------------------
# Generate Medications
# -----------------------------
medication_data = []

# Medications for all visits
for idx, row in df_visits.iterrows():
    num_meds = random.randint(0,3)
    for _ in range(num_meds):
        medication_data.append({
            'PatientID': row['PatientID'],
            'VisitID': row['VisitID'],
            'MedicationID': f'Med_{random.randint(1, NUM_MEDICATIONS):03d}',
            'Dosage': f'{random.randint(1,3)} tablets/day'
        })

# Medications for operations
for idx, row in df_operations.iterrows():
    num_meds = random.randint(1,5)
    for _ in range(num_meds):
        medication_data.append({
            'PatientID': row['PatientID'],
            'OperationID': row['OperationID'],
            'MedicationID': f'Med_{random.randint(1, NUM_MEDICATIONS):03d}',
            'Dosage': f'{random.randint(1,3)} tablets/day'
        })

df_medications = pd.DataFrame(medication_data)

# -----------------------------
# Generate Billing
# -----------------------------
billing_data = []

# Billing for visits
for idx, row in df_visits.iterrows():
    billing_data.append({
        'PatientID': row['PatientID'],
        'VisitID': row['VisitID'],
        'Amount': row['DoctorVisitFee'] + random.randint(500,5000),
        'PaymentStatus': random.choice(['Paid','Pending','Insurance'])
    })

# Billing for operations
for idx, row in df_operations.iterrows():
    billing_data.append({
        'PatientID': row['PatientID'],
        'OperationID': row['OperationID'],
        'Amount': random.randint(5000,50000),
        'PaymentStatus': random.choice(['Paid','Pending','Insurance'])
    })

df_billing = pd.DataFrame(billing_data)

# -----------------------------
# Save CSVs
# -----------------------------
df_patients.to_csv('Dim_Patient.csv', index=False)
df_visits.to_csv('Fact_Visits.csv', index=False)
df_operations.to_csv('Fact_Operations.csv', index=False)
df_labtests.to_csv('Fact_LabTests.csv', index=False)
df_medications.to_csv('Fact_Medications.csv', index=False)
df_billing.to_csv('Fact_Billing.csv', index=False)

print("CSV files generated successfully!")
