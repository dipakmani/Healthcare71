import pandas as pd
import numpy as np
from faker import Faker

fake = Faker()
np.random.seed(42)

# -----------------------------
# 1️⃣ Parameters
# -----------------------------
num_unique_patients = 200000
num_visits = 500000
num_doctors = 100
num_labtests = 50
num_procedures = 50
num_equipment = 30
num_medications = 200

# -----------------------------
# 2️⃣ Generate Dim_Patient
# -----------------------------
patient_ids = [f'Patient_{i:06d}' for i in range(1, num_unique_patients+1)]
patient_data = []

for pid in patient_ids:
    patient_data.append({
        'PatientID': pid,
        'FirstName': fake.first_name(),
        'LastName': fake.last_name(),
        'Gender': np.random.choice(['M', 'F']),
        'DOB': fake.date_of_birth(minimum_age=0, maximum_age=90),
        'City': fake.city(),
        'State': fake.state(),
        'Country': 'India',
        'BloodGroup': np.random.choice(['A+', 'A-', 'B+', 'B-', 'AB+', 'AB-', 'O+', 'O-'])
    })

df_patients = pd.DataFrame(patient_data)

# -----------------------------
# 3️⃣ Generate Fact_Visits
# -----------------------------
visit_ids = [f'Visit_{i:06d}' for i in range(1, num_visits+1)]
assigned_patients = np.random.choice(patient_ids, size=num_visits, replace=True)
doctor_ids = [f'Doctor_{i:03d}' for i in range(1, num_doctors+1)]
visit_dates = pd.to_datetime(np.random.randint(
    pd.Timestamp('2020-01-01').value//10**9,
    pd.Timestamp('2025-01-01').value//10**9,
    num_visits), unit='s')
doctor_fees = np.random.randint(100, 1501, size=num_visits)

df_visits = pd.DataFrame({
    'VisitID': visit_ids,
    'PatientID': assigned_patients,
    'DoctorID': np.random.choice(doctor_ids, size=num_visits, replace=True),
    'VisitDate': visit_dates,
    'DoctorFee': doctor_fees
})

# Merge patient attributes
df_visits = df_visits.merge(df_patients, on='PatientID', how='left')

# -----------------------------
# 4️⃣ Assign operations to 50% unique patients
# -----------------------------
operation_patients = np.random.choice(patient_ids, size=num_unique_patients//2, replace=False)
df_visits['OperationFlag'] = df_visits['PatientID'].apply(lambda x: 1 if x in operation_patients else 0)

# -----------------------------
# 5️⃣ Generate Fact_Operations
# -----------------------------
operations_data = []
operation_ids = []
for pid in operation_patients:
    # Get all visits for patient
    patient_visits = df_visits[df_visits['PatientID'] == pid]
    # Choose one visit for operation
    visit_row = patient_visits.sample(1).iloc[0]
    op_date = visit_row['VisitDate'] + pd.to_timedelta(np.random.randint(1,15), unit='d')
    discharge_date = op_date + pd.to_timedelta(np.random.randint(1,10), unit='d')
    procedure_id = f'Procedure_{np.random.randint(1, num_procedures+1):03d}'
    equipment_used = [f'Equipment_{np.random.randint(1, num_equipment+1):02d}' for _ in range(np.random.randint(1,4))]
    operation_id = f'Operation_{pid}'
    operation_ids.append(operation_id)
    
    operations_data.append({
        'OperationID': operation_id,
        'PatientID': pid,
        'VisitID': visit_row['VisitID'],
        'OperationDate': op_date,
        'DischargeDate': discharge_date,
        'ProcedureID': procedure_id,
        'EquipmentUsed': ','.join(equipment_used)
    })

df_operations = pd.DataFrame(operations_data)

# -----------------------------
# 6️⃣ Generate Fact_LabTests for operations
# -----------------------------
labtests_data = []
labtest_ids = [f'LabTest_{i:03d}' for i in range(1, num_labtests+1)]
for idx, row in df_operations.iterrows():
    num_tests = np.random.randint(1,6)
    tests = np.random.choice(labtest_ids, size=num_tests, replace=False)
    for test in tests:
        labtests_data.append({
            'PatientID': row['PatientID'],
            'OperationID': row['OperationID'],
            'LabTestID': test,
            'Result': np.random.choice(['Normal', 'High', 'Low'])
        })
df_labtests = pd.DataFrame(labtests_data)

# -----------------------------
# 7️⃣ Generate Fact_Medications
# -----------------------------
medication_ids = [f'Medication_{i:03d}' for i in range(1, num_medications+1)]
medications_data = []
for idx, row in df_visits.iterrows():
    num_meds = np.random.randint(0,6)  # Some visits may have 0 meds
    meds = np.random.choice(medication_ids, size=num_meds, replace=False)
    for med in meds:
        medications_data.append({
            'PatientID': row['PatientID'],
            'VisitID': row['VisitID'],
            'MedicationID': med,
            'Dosage': f"{np.random.randint(1,4)} times/day",
            'Quantity': np.random.randint(1,21)
        })
df_medications = pd.DataFrame(medications_data)

# -----------------------------
# 8️⃣ Generate Fact_Billing
# -----------------------------
billing_data = []
for idx, row in df_visits.iterrows():
    amount = row['DoctorFee']
    if row['OperationFlag'] == 1:
        amount += np.random.randint(5000, 50001)  # operation cost
    billing_data.append({
        'PatientID': row['PatientID'],
        'VisitID': row['VisitID'],
        'Amount': amount,
        'PaymentStatus': np.random.choice(['Paid', 'Pending', 'Insurance']),
        'InsuranceProvider': np.random.choice(['Provider A', 'Provider B', 'Provider C', 'None'])
    })
df_billing = pd.DataFrame(billing_data)

# -----------------------------
# ✅ Save CSVs
# -----------------------------
df_patients.to_csv('Dim_Patient.csv', index=False)
df_visits.to_csv('Fact_Visits.csv', index=False)
df_operations.to_csv('Fact_Operations.csv', index=False)
df_labtests.to_csv('Fact_LabTests.csv', index=False)
df_medications.to_csv('Fact_Medications.csv', index=False)
df_billing.to_csv('Fact_Billing.csv', index=False)

print("Data generation completed! CSVs ready for use.")
