# Exeter Diabetes codelists
CPRD Aurum codelists from the Exeter Diabetes team

&nbsp;

## General notes on implementation
* Only codes with 'valid' dates used unless otherwise stated ('any date'). A valid date is an obsdate (for medcodes), issuedate (for prodcodes), epistart (for ICD10 codes) or evdate (for OPCS4 codes) which is no earlier than the patient's date of birth (no earlier than the month of birth if date of birth is not available; no earlier than full date of birth if this is available), no later than the patient's date of death (earliest of cprd_ddeath (Patient table) and dod/dor where dod not available (ONS death data)) where this is present, no later than deregistration where this is present, and no later than the last collection date from the Practice.
* Patient date of birth = 15th of month of birth (where month of birth and year of birth available), and 1st July where only year of birth available.
* Biomarker tests before 1990 removed as most not developed by then.

&nbsp;

## Biomarker algorithms
* Values outside of acceptable limits removed (Biomarkers/biomarker_acceptable_limits; note this table is suitable for ADULT height, weight and BMI measurements only)
* Only values with acceptable units (numunitid) included (Biomarkers/biomarker_acceptable_units; includes missing numunitid for all biomarkers)
* If multiple biomarker values on the same day, the mean is used (including for HbA1c, after conversion to the same unit code)

&nbsp;

### BMI
* Take coded BMI value
* If no coded value, where height and weight recorded on same obsdate, calculate bmi
* Note: this only applies for adults/ over 18s

&nbsp;

### eGFR (estimated glomerular filtration rate)
*

&nbsp;

### ACR (Albumin Creatinine Ratio)
* Take coded ACR value
* If no coded value, where urine albumin and creatinine measurements recorded on same obsdate, calculate acr

&nbsp;

## Co-morbidity algorithms
* Earliest (valid) medcode/ICD10 code/OPCS4 code date used as date of diagnosis

&nbsp;

### CKD (Chronic Kidney Disease)
*

&nbsp;

## Diabetes algorithms

### Defining a cohort of mixed Type 1 and Type 2 diabetes
Include participants with:
* 'acceptable' patient flag (Patient table) = 1
* At least one diabetes QOF (Quality and Outcomes Framework) medcode (Diabetes/exeter_medcodelist_qof_diabetes)
* No diabetes exclusion medcodes (codes for non-Type 1/non-Type 2 diabetes mellitus; Diabetes/exeter_medcodelist_exclusion_diabetes) with any date

&nbsp;

### Defining the diagnosis date of diabetes
Diagnosis date is earliest of:
* Any diabetes medcode (Diabetes/exeter_medcodelist_all_diabetes) except those with obstypeid=4 (family history)
* A prescription for glucose lowering medication including insulin (Diabetes medications/exeter_prodcodelist_ohas and Diabetes medications/exeter_prodcodelist_insulin)
* Any HbA1c (Biomarkers/exeter_medcodelist_hba1c) >=47.5 mmol/mol (values <=20 assumed to be in % units and converted to mmol/mol)


&nbsp;

### Defining Type 1 vs Type 2 in a mixed Type 1/Type 2 cohort
* No insulin prescriptions: Type 2
* With at least one insulin prescription:
  * With multiple Type 1- and/or Type 2-specific medcodes with any date (exeter_medcodelist_all_diabetes, category="type 1" or "type 2"):
    * If number of Type 1 medcodes >=2 x number of Type 2 medcodes, Type 1, otherwise Type 2
  * With one Type 1- or Type 2-specific medcode with any date:
    * Patient is categorise according to this medcode
  * With no one Type 1- or Type 2-specific medcodes:
    * If time between diagnosis date and start of insulin treatment is available (i.e. diagnosis date >= start of registration):
      * If diagnosed <35 years of age and on insulin within 1 year of diagnosis, Type 1, otherwise Type 2
    * If time between diagnosis date and start of insulin treatment is not available (i.e. diagnosis date < start of registration):
      * If diagnosed <35 years and not currently taking a non-insulin glucose lowering medication (no prescription for a non-insulin glucose lowering medication within 6 months of end of records (earliest of death/deregistration/last collection date from Practice)), Type 1, otherwise Type 2
      
&nbsp;

## Sociodemographics algorithms

### Ethnicity
* Take most commonly recorded ethnicity category (eth5, or eth16, excluding all 'unknown' ethnicity codes)
* If more than one most commonly recorded ethnicity, take most recently recorded ethnicity category
* Everyone else categorised as unknown

&nbsp;

### Smoking
* Take most commonly recorded smoking category (non-smoker, ex-smoker, active smoker)
* If most recent is 'non-smoker' but have previously been recorded as active smoker, then categorise as 'ex-smoker'
* Look next most recent date if more than one category is coded on most recent date

&nbsp;

### Alcohol consumption
* Take most recently recorded alcohol consumption level (0 = none, 1 = within limits, 2 = excess, 3 = harmful)
* If ever coded with a level3 code then categorise as level3
* If multiple levels coded on most recent date, take the highest level

&nbsp;

