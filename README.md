# Hospital Patient Records Analysis
## Table of Content

* [Case Study](#case-study)
* [Dataset Description](#dataset-description)
* [ER Diagram](#er-diagram)
* [Data Cleaning](#data-cleaning)
* [Dashboard](#dashboard)

## Case Study
For the Maven Hospital Challenge, you’ll play the role of an Analytics Consultant for Massachusetts General Hospital (MGH).

You’ve been asked to build a high-level KPI report for the executive team, based on a subset of patient records. The purpose of the report is to give stakeholders visibility into the hospital’s recent performance, and answer the following questions:

- How many patients have been admitted or readmitted over time?
- How long are patients staying in the hospital, on average?
- How much is the average cost per visit?
- How many procedures are covered by insurance?

The dashboard should scale to accommodate new data over time, but the CEO has asked you to summarize any insights you can derive from the sample provided.
  
## Dataset Description
Our data set consists of the following observations which include:

#### Encounters:
- **Id**: Primary Key. Unique Identifier of the encounter.
- **Start**: The date and time (iso8601 UTC Date (yyyy-MM-dd'T'HH:mm'Z')) the encounter started.
- **Stop**: The date and time (iso8601 UTC Date (yyyy-MM-dd'T'HH:mm'Z')) the encounter concluded.
- **Patient**: Foreign key to the Patient.
- **Organization**: Foreign key to the Organization.
- **Payer**: Foreign key to the Payer.
- **EncounterClass**: The class of the encounter, such as ambulatory, emergency, inpatient, wellness, or urgent care.
- **Code**: Encounter code from SNOMED-CT.
- **Description**: Description of the type of encounter.
- **Base_Encounter_Cost**: The base cost of the encounter, not including any line item costs related to medications, immunizations, procedures, or other services.
- **Total_Claim_Cost**: The total cost of the encounter, including all line items.
- **Payer_Coverage**: The amount of cost covered by the Payer.
- **ReasonCode**: Diagnosis code from SNOMED-CT, only if this encounter targeted a specific condition.
- **ReasonDescription**: Description of the reason code.
#### Organization:
- **Id**: Primary key of the Organization.
- **Name**: Name of the Organization.
- **Address**: Organization's street address without commas or newlines.
- **City**: Street address city.
- **State**: Street address state abbreviation.
- **Zip**: Street address zip or postal code.
- **Lat**: Latitude of Organization's address.
- **Lon**: Longitude of Organization's address.
#### Patients:
- **Id**: Primary Key. Unique Identifier of the patient.
- **BirthDate**: The date (YYYY-MM-DD) the patient was born.
- **DeathDate**: The date (YYYY-MM-DD) the patient died.
- **Prefix**: Name prefix, such as Mr., Mrs., Dr., etc.
- **First**: First name of the patient.
- **Middle**: Middle name of the patient.
- **Last**: Last or surname of the patient.
- **Suffix**: Name suffix, such as PhD, MD, JD, etc.
- **Maiden**: Maiden name of the patient.
- **Marital**: Marital Status. M is married, S is single. Currently no support for divorce (D) or widowing (W).
- **Race**: Description of the patient's primary race.
- **Ethnicity**: Description of the patient's primary ethnicity.
- **Gender**: Gender. M is male, F is female.
- **BirthPlace**: Name of the town where the patient was born.
- **Address**: Patient's street address without commas or newlines.
- **City**: Patient's address city.
- **State**: Patient's address state.
- **County**: Patient's address county.
- **FIPS County Code**: Patient's FIPS county code.
- **Zip**: Patient's zip code.
- **Lat**: Latitude of Patient's address.
- **Lon**: Longitude of Patient's address.
#### Payers:
- **Id**: Primary key of the Payer (e.g., Insurance).
- **Name**: Name of the Payer.
- **Address**: Payer's street address without commas or newlines.
- **City**: Street address city.
- **State_Headquartered**: Street address state abbreviation.
- **Zip**: Street address zip or postal code.
- **Phone**: Payer's phone number.
#### Procedures:
- **Start**: The date and time (iso8601 UTC Date (yyyy-MM-dd'T'HH:mm'Z')) the procedure was performed.
- **Stop**: The date and time (iso8601 UTC Date (yyyy-MM-dd'T'HH:mm'Z')) the procedure was completed, if applicable.
- **Patient**: Foreign key to the Patient.
- **Encounter**: Foreign key to the Encounter where the procedure was performed.
- **Code**: Procedure code from SNOMED-CT.
- **Description**: Description of the procedure.
- **Base_Cost**: The line item cost of the procedure.
- **ReasonCode**: Diagnosis code from SNOMED-CT specifying why this procedure was performed.
- **ReasonDescription**: Description of the reason code.

## ER Diagram
![Hospital Records drawio](https://github.com/karlyndiary/Hospital-Patient-Records-Analysis/assets/116041695/eff333ef-85b6-4fd2-bcc2-f297010bfbb5)

## Data Cleaning
### Calculated Fields
- Admitted or Readmitted Metric
```
{ FIXED [Start] : COUNTD([Patient]) }
```
- Stay Duration
```
DATEDIFF('day', [Start], [Stop])
```
- Count of Procedures
```
IF CONTAINS([Description], "procedure") THEN TRUE ELSE FALSE END
```
- Procedure Coverage Status
```
IF [Count Procedures] THEN
    IF [Payer Coverage] > 0 THEN "Covered by Insurance" ELSE "Not Covered by Insurance" END
ELSE
    "Not a Procedure"
END
```
- Age
```
DATEDIFF('year', [Birthdate], TODAY())
```
- Age Category
```
IF [Age] >= 0 AND [Age] < 10 THEN "0-9"
ELSEIF [Age] >= 10 AND [Age] < 20 THEN "10-19"
ELSEIF [Age] >= 20 AND [Age] < 30 THEN "20-29"
ELSEIF [Age] >= 30 AND [Age] < 40 THEN "30-39"
ELSEIF [Age] >= 40 AND [Age] < 50 THEN "40-49"
ELSEIF [Age] >= 50 AND [Age] < 60 THEN "50-59"
ELSEIF [Age] >= 60 AND [Age] < 70 THEN "60-69"
ELSEIF [Age] >= 70 AND [Age] < 80 THEN "70-79"
ELSEIF [Age] >= 80 AND [Age] < 90 THEN "80-89"
ELSE "90+"
END
```
- Admission Status
```
IF [Count Procedures] = True THEN
    "Admitted"
ELSE
    "Not Admitted"
END
```
- Latest Encounter Date
```
{ FIXED [Patient] : MAX([Start]) }
```
- Procedure Coverage Status
```
IF [Count Procedures] THEN
    IF [Payer Coverage] > 0 THEN "Covered by Insurance" ELSE "Not Covered by Insurance" END
ELSE
    "Not a Procedure"
END
```
- Procedure Coverage Status
```
IF [Count Procedures] THEN
    IF [Payer Coverage] > 0 THEN "Covered by Insurance" ELSE "Not Covered by Insurance" END
ELSE
    "Not a Procedure"
END
```
- Filter to the Latest Date
```
IF [Start] = [Latest Encounter Date] THEN 1 ELSE 0 END
```
- Is Alive?
```
IF ISNULL([DeathDate]) THEN 'Alive' ELSE 'Deceased' END
```
- Cleaning first name and last name
```
REGEXP_REPLACE([first], '[^a-zA-Z]', '')
REGEXP_REPLACE([last], '[^a-zA-Z]', '')
```
- Full name
```
IFNULL([Prefix], '') + ' ' + [First_name] + ' ' + [Last_name]
```

## Dashboard
### Overview 
![Overview](https://github.com/user-attachments/assets/4110571a-ea05-4a39-9f4a-a4d2297842d0)

### Patients Detail 
![Detail](https://github.com/user-attachments/assets/6e0c2ca5-398b-442f-8c1c-dd4eb329ba7a)
