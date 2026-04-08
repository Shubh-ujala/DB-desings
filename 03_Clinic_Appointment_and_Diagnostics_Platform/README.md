# Clinic Appointment and Diagnostics Platform - DB Design

This folder contains the database design for a clinic platform that manages:

- User accounts for doctors and patients
- Doctor profiles and clinic mapping
- Patient medical profile basics
- Appointment booking and consultation flow
- Diagnostic test and test report lifecycle
- Appointment-level payments

## Exported Diagram

- Diagram file: `diagram-export-4-8-2026-8_25_10-PM.png`

## Core Modules

1. Identity and Access
2. Clinic and Doctor Management
3. Patient Management
4. Appointment and Consultation
5. Diagnostics and Reporting
6. Billing and Payments

## Entities

### 1) `user`
Base account table used for authentication and role separation.

Fields:
- `id` (PK)
- `name`
- `email`
- `password`
- `avatar_img`
- `role` (`patient`, `doctor`)
- `createdAt`, `updatedAt`

### 2) `doctor`
Doctor-specific profile mapped to a user account and clinic.

Fields:
- `id` (PK)
- `user_id` (FK -> `user.id`)
- `clinic_id` (FK -> `clinic.id`)
- `specialization`
- `experience_years`
- `bio`
- `is_available`
- `available_days`
- `available_timeslots`
- `createdAt`, `updatedAt`

### 3) `patient`
Patient-specific profile mapped to a user account.

Fields:
- `id` (PK)
- `user_id` (FK -> `user.id`)
- `full_name`
- `father_name`
- `blood_group`
- `phone_number`
- `alternate_phone_number`
- `issues`
- `createdAt`, `updatedAt`

### 4) `clinic`
Clinic master table.

Fields:
- `id` (PK)
- `clinic_name`
- `clinic_location`
- `clinic_timings`
- `is_open`
- `createdAt`, `updatedAt`

### 5) `appointment`
Appointment slots between a patient and doctor at a clinic.

Fields:
- `id` (PK)
- `doctor_id` (FK -> `doctor.id`)
- `patient_id` (FK -> `patient.id`)
- `clinic_id` (FK -> `clinic.id`)
- `number_patients`
- `max_patients`
- `appointment_date`
- `consultation_fees`
- `createdAt`, `updatedAt`

### 6) `consultation`
Medical consultation details captured for a booked appointment.

Fields:
- `id` (PK)
- `doctor_id` (FK -> `doctor.id`)
- `patient_id` (FK -> `patient.id`)
- `appointment_id` (FK -> `appointment.id`)
- `diagnosis`
- `prescription`
- `precautions`
- `createdAt`, `updatedAt`

### 7) `diagnostic_test`
Diagnostic tests prescribed during a consultation.

Fields:
- `id` (PK)
- `consultation_id` (FK -> `consultation.id`)
- `test_name`
- `test_price`
- `createdAt`, `updatedAt`

### 8) `test_report`
Diagnostic test result records for patients.

Fields:
- `id` (PK)
- `diagnostic_test_id` (FK -> `diagnostic_test.id`)
- `patient_id` (FK -> `patient.id`)
- `test_report_result`
- `test_report_summary`
- `report_file`
- `report_generation_date`
- `createdAt`, `updatedAt`

### 9) `payment`
Payment records tied to appointments.

Fields:
- `id` (PK)
- `appointment_id` (FK -> `appointment.id`)
- `amount`
- `payment_method` (`Cash`, `Online`, `EMI`)
- `payment_status` (`pending`, `success`, `failed`)
- `createdAt`, `updatedAt`

## Relationships

- One `user` -> one `doctor` (for doctor accounts)
- One `user` -> one `patient` (for patient accounts)
- One `clinic` -> many `doctor`
- One `doctor` -> many `appointment`
- One `patient` -> many `appointment`
- One `clinic` -> many `appointment`
- One `appointment` -> one `consultation`
- One `consultation` -> many `diagnostic_test`
- One `diagnostic_test` -> one `test_report`
- One `appointment` -> many `payment` (or one, based on business rule)

## Suggested Constraints

- Add `UNIQUE (email)` in `user`
- Add `CHECK (experience_years >= 0)` in `doctor`
- Add `CHECK (consultation_fees >= 0)` in `appointment`
- Add `CHECK (test_price >= 0)` in `diagnostic_test`
- Add `CHECK (amount >= 0)` in `payment`
- Add `CHECK (number_patients <= max_patients)` in `appointment`
- Add `UNIQUE (appointment_id)` in `consultation` if strictly one consultation per appointment

## Suggested Indexes

- `user(email)`
- `doctor(user_id, clinic_id)`
- `patient(user_id, phone_number)`
- `appointment(doctor_id, appointment_date)`
- `appointment(patient_id, appointment_date)`
- `consultation(appointment_id)`
- `diagnostic_test(consultation_id)`
- `test_report(diagnostic_test_id, patient_id)`
- `payment(appointment_id, createdAt)`

## Notes

- In the raw draft, a few names are misspelled (`Clinc`, `expreriance_years`, etc.); this README uses normalized naming for implementation.
- You can keep table names singular (as written here) or convert to plural, but keep naming consistent across all migrations.
- This schema is a strong base for a clinic ERP-lite system and can be extended with:
	- doctor leave/holiday calendars
	- multi-branch clinics
	- insurance claims
	- digital prescription templates
