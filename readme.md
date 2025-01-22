# Exeter Diabetes codelists
CPRD Aurum codelists from the Exeter Diabetes team. These include medcodes (for use with the Observation table) and prodcodes (for use with the Drug Issue table), as well as ICD10 and OPCS4 codelists for use with linked HES APC data. All codelists have been clinically-reviewed except a small subset where review by a non-clinician was deemed acceptable (BMI, weight, height and blood pressure readings) or where PBCL biomarker codes were used (see [Code list generation process](https://github.com/Exeter-Diabetes/CPRD-Codelists/blob/main/readme.md#code-list-generation-process) below).

We have also included Read and SNOMED codelists created during medcode list development (see [Code list generation process](https://github.com/Exeter-Diabetes/CPRD-Codelists/blob/main/readme.md#code-list-generation-process) below); where available, these are located in each of the subfolders in the [Medcodes](https://github.com/Exeter-Diabetes/CPRD-Codelists/tree/main/Medcodes) folder. Note that Read and SNOMED codelists are csv files and may have hash (#) symbols as suffixes in the 'readcode'/'snomedcode'/'medcodeid' columns to prevent rounding when the csv files are opened in Excel.

In 2024, we updated some of our algorithms and codelists (main branch of this repository), in addition to adding new codelists. Papers published by the Exeter Diabetes team in or prior to 2024 use the algorithms and codelists in the 'pre-2024' branch of this repository, all of which were based on the October 2020 extract of CPRD Aurum. Where we have updated codelists since then, we have added in additional codes which appear in the June 2024 extract of CPRD Aurum (where these exist), but haven't removed codes which were in the October 2020 but not the June 2024 extract; thus the codelists are compatible with either extract. New medcodes were found by rejoining the original validated Read and SNOMED codelists (see [Medcode generation process](https://github.com/Exeter-Diabetes/CPRD-Codelists/tree/main?tab=readme-ov-file#medcodes)) with the June 2024 CPRD Aurum Medical Dictionary, or by searching for key words/terms in the June 2024 CPRD Aurum Medical Dictionary (whichever method was used to create the original 2020 codelist). Any new codes found were reviewed before addition. Medcode lists which have been updated are: 'all diabetes', 'QOF diabetes', diabetes insipidus (not used pre-2024), family history of diabetes and premature CVD, all biomarkers, all comorbidities except care home, hearing loss, pulmonary embolism, deep vein thrombosis, surgical pancreatic resection and pancreatic insufficiency (NB: weight loss and polydipsia only added in 2024 so may not contain all codes from October 2020). Some codelists have also changed in their scope due to changes in the algorithms they are used with: 'all diabetes' now contains all types of diabetes mellitus (previously, codes for non-type 1/type 2 diabetes were in an 'exclusion diabetes' codelist which also included diabetes insipidus; the latter is now a separate codelist). Prodcode lists were updated by searching for known generic and brand names in the June 2024 CPRD Aurum Product Dictionary (some lists available in this repository at Prodcode lists/Drug name lists/). Prodcode lists which have been updated are the four blood pressure medication codelists ('ace_inhibitors', 'beta_blockers', 'calcium_channel_blockers', and 'arb'), some diabetes medications ('ohas' and 'insulin'), statins, and finerenone (in Renal medications; added in 2024 so may not contain all codes from October 2020).

Our scripts which use these codelists and the below algorithms to define cohorts and pull in relevant biomarker/comorbidity/sociodemographic information from CPRD Aurum can be found in our [CPRD-Cohort-scripts repository](https://github.com/Exeter-Diabetes/CPRD-Cohort-scripts).

&nbsp;

## General notes on implementation
* Only codes with 'valid' dates used unless otherwise stated ('any date'). A valid date is an obsdate (for medcodes), issuedate (for prodcodes), epistart (for ICD10 codes) or evdate (for OPCS4 codes) which is no earlier than the patient's minimum date of birth (no earlier than the month of birth if date of birth is not available; no earlier than full date of birth if this is available), no later than deregistration where this is present  (for obsdates/issuedates), no later than the last collection date from the Practice (for obsdates/issuedates), and no later than the end of linkage data (for epistart dates/evdates).
* Obsdate (for medcodes) and issuedate (for prodcodes) were used rather than using 'enterdate' in either the Observation or Drug Issue table. We looked at an example of a condition (diabetes) and a biomarker (HbA1c) and found that enterdate was earlier than obsdate for <0.1% of observations, once clearly incorrect enterdates (01/01/1860, 30/12/1899 and 31/12/1899) were removed, so it would not make a significant difference to use e.g. the minimum of enterdate and obsdate compared to using obsdate alone. Looking at a medication example (non-insulin diabetes medications), we found that enterdate was earlier than issuedate in ~10% of cases (and did not contain incorrect pre-1900 values), and was frequently a whole number of weeks (e.g. 7, 14, or 21 days), suggesting a genuine time difference between when the prescription was entered onto the computer system and when it was issued to the patient. For diabetes diagnosis we might be more interested in when the clinician prescribed the medication (i.e. the enterdate where earlier than issuedate) than when the patient received it, whereas for our treatment selection work we are interested in when the patient started taking the medication. Since very few patients are diagnosed based on a diabetes medication script, only ~10% of scripts have an issuedate later than the enterdate, and for this 10% the median time from enterdate to issuedate was 21 days, we used issuedate throughout our work.
* Patient date of birth = earliest of :
  * 15th of month of birth (where month of birth and year of birth available) / 30th June where only year of birth available
  * Earliest medcode in Observation table if this is earlier (excluding medcodes before mob/yob)
  * Registration start date (excluding if before mob/yob)

&nbsp;

## Biomarker algorithms
* Values outside of acceptable limits removed (Biomarkers/biomarker_acceptable_limits; note this table is suitable for ADULT height, weight and BMI measurements only)
* Only values with acceptable units (numunitid) included (Biomarkers/biomarker_acceptable_units; includes missing numunitid for all biomarkers)
* If multiple biomarker values on the same day, the mean is used (including for HbA1c, after conversion to the same unit code)

&nbsp;

### ACR (Albumin Creatinine Ratio)
* Preferentially use coded ACR value
* If no coded value, where urine albumin and urine creatinine measurements recorded on same obsdate, calculate ACR

&nbsp;

### BMI
* Preferentially use coded BMI value
* If no coded value, where height and weight recorded on same obsdate, use these to calculate BMI (for adults only)

&nbsp;

### eGFR (estimated glomerular filtration rate)
* Use cleaned serum creatinine readings, age at reading and sex to calculate eGFR as per CKD-EPI Creatinine Equation 2021 (https://www.kidney.org/professionals/kdoqi/gfr_calculator/formula; ethnicity not used in this equation)

&nbsp;


### Haematocrit
* Convert all to proportion by dividing those >1 by 100

&nbsp;

### Haemoglobin
* Convert all to g/L (some in g/dL) by multiplying values <30 by 10

&nbsp;

### HbA1c
* Remove if before 1990 as HbA1c not widely used before then
* Convert all to mmol/mol by converting values <=20 (assume these are in % units)

&nbsp;

## Comorbidity algorithms
* Earliest (valid) medcode/ICD10 code/OPCS4 code date used as date of diagnosis

&nbsp;

### CKD (Chronic Kidney Disease) stage
* Use eGFR measurements calculated as above
* Convert eGFR measurements to CKD stage (1/2/3a/3b/4/5 as per https://www.kidney.org/atoz/content/gfr)
* Only keep CKD stages confirmed by multiple readings separated by >=90 days (without a measurement corresponding to a different stage within this 90 day period). This is to exclude readings corresponding to acute kidney injury (AKI).
* Remove any instances where patients appear to regress to a less severe CKD stage e.g. if they move from stage 4 to 3b, remove the 3b measurements
* Diagnosis date for CKD stage is the earliest eGFR measurement corresponding to this stage once values have been cleaned as above.
* CKD stage 5 (end stage renal disease [ESRD]) is defined using eGFR as above OR by the presence of a CKD5 medcode/ICD10/OPCS4 code

&nbsp;

### Frailty

Note that the 'frailty_simple' codelist is not exhaustive, but contains three of the most commonly used medcodes for frailty, classified by frailty severity.

&nbsp;

## Diabetes algorithms

A flow diagram of the overall process of defining a diabetes cohort can be found in our [CPRD-Cohort-scripts repository](https://github.com/Exeter-Diabetes/CPRD-Cohort-scripts#introduction).

### Defining a cohort of mixed Type 1/Type 2/'other' diabetes
Include participants with:
* At least one diabetes QOF (Quality and Outcomes Framework) medcode (Diabetes/exeter_medcodelist_qof_diabetes). The QOF codelist was constructed from Read codes from version 38 and SNOMED codes from version 44 of the QOF, which include all codes from previous versions. This includes QOF codes for non-Type 1/non-Type 2 diabetes mellitus (note that gestational diabetes is not in QOF).
* No gestational diabetes codes: we exclude patients with gestational diabetes as it would require a more complex algorithm to calculate their diagnosis dates.
* No code for diabetes insipidus: we exclude patients with diabetes insipidus as they sometimes incorrectly have diabetes mellitus codes. Note that we do not exclude individuals with codes for syndromes which include diabetes insipidus (codes for these syndromes are not included in the diabetes insipidus codelist), but only with standalone diabetes insipidus codes.

&nbsp;

### Defining the diagnosis date of diabetes
Diagnosis date is earliest of:
* Any diabetes medcode (from the 'all diabetes' codelist) except those with obstypeid=4 (family history), codes for diabetes in pregnancy where it is not clear if this is pre-existing or gestational, or those in year of birth for those with Type 2 diabetes (see Type 1/Type 2 classification algorithm below)
* A prescription for glucose lowering medication including insulin (Diabetes medications/exeter_prodcodelist_ohas and Diabetes medications/exeter_prodcodelist_insulin)
* Any HbA1c (Biomarkers/exeter_medcodelist_hba1c) >=47.5 mmol/mol (values <=20 assumed to be in % units and converted to mmol/mol)

Type 2 patients (see Type 1/Type 2 classification algorithm below) with a prescription for a glucose-lowering medication or HbA1c >=47.5 mmol/mol in their year of birth, or with no diabetes medcodes/prescriptions for glucose-lowering medications/HbA1cs >=47.5 mmol/mol after the year of birth are excluded from further analysis as presumably there are coding errors.

Diagnosis dates within -30 to +90 days (inclusive) of registration start are set to missing as they may be unreliable (similar to https://bmjopen.bmj.com/content/7/10/e017989 except we also excluded diagnoses up to 30 days before registration as our analysis showed an excess of diagnoses in this period).

&nbsp;

### Defining diabetes type

Those with any diabetes exclusion medcode (codes for non-Type 1/non-Type 2 diabetes mellitus; Diabetes/exeter_medcodelist_exclusion_diabetes) with any date are defined as having 'Other' diabetes type (i.e. not Type 1 or 2) and require further processing to ascertain diabetes type.

For those with no diabetes exclusion medcodes:
* No insulin prescriptions: Type 2
* With at least one insulin prescription:
  * With multiple Type 1- and/or Type 2-specific medcodes with any date (exeter_medcodelist_all_diabetes, category="type 1" or "type 2"):
    * If number of Type 1 medcodes >=2 x number of Type 2 medcodes, Type 1, otherwise Type 2
  * With one Type 1- or Type 2-specific medcode with any date:
    * Patient is categorised according to this medcode
  * With no one Type 1- or Type 2-specific medcodes:
    * If time between diagnosis date and start of insulin treatment is available (i.e. diagnosis date >= start of registration):
      * If diagnosed <35 years of age and on insulin within 1 year of diagnosis, Type 1, otherwise Type 2
    * If time between diagnosis date and start of insulin treatment is not available (i.e. diagnosis date < start of registration):
      * If diagnosed <35 years and not currently taking a non-insulin glucose lowering medication (no prescription for a non-insulin glucose lowering medication within 6 months of end of records (earliest of death/deregistration/last collection date from Practice)), Type 1, otherwise Type 2
      
Since diabetes type is used to inform whether diabetes medcodes in year of birth should be excluded or not (see above 'Defining the diagnosis date of diabetes'), this can create a circular problem in those with no Type 1 or Type 2-specific codes, where someone is classified as Type 1 if diabetes medcodes in year of birth are included, and Type 2 if diabetes medcodes in year of birth are excluded (or vice versa, if time to insulin from diagnosis is affected). These people are classed as 'unclassified', and need further investigation to establish diabetes type and date/age of diagnosis (we have excluded from our general Type 1/Type 2 cohort).

See our paper https://linkinghub.elsevier.com/retrieve/pii/S0895-4356(22)00272-4 for how this compares to other methods for classifying Type 1 and Type 2 diabetes.
      
&nbsp;

## Sociodemographics algorithms

### Ethnicity
* Three different ethnicity variables are produced: 5-category (0=White, 1=South Asian, 2=Black, 3=Other, 4=Mixed, 5=Unknown), 16-category (1=White British, 2=White Irish, 3=Other White, 4=White and Black Caribbean, 5=White and Black African, 6=White and Asian, 7=Other Mixed, 8=Indian, 9=Pakistani, 10=Bangladeshi, 11=Other Asian, 12=Caribbean, 13=African, 14=Other Black, 15=Chinese, 16=Other, 17=Unknown), and QRISK2-category (1=White, 2=Indian, 3=Pakistani, 4=Bangladeshi, 5=Other Asian, 6=Black Caribbean, 7=Black African, 8=Chinese, 9=Other). Each is determined separately by:
  * Taking the most commonly recorded ethnicity category, excluding all 'unknown' ethnicity codes
  * If more than one most commonly recorded ethnicity category, use the most recently recorded. If multiple different ethnicities recorded on the most recent date, categorise as unknown
* For each person, if the 16-category or QRISK2-category category conflicts with the 5-category ethnicity\*, the 16-/QRISK2-category ethnicity is set to unknown
* For any unknown ethnicities, secondary care [HES] ethnicity is used where available and where is identical to 5-category ethncity (or where 5-category ethnicity missing)

&nbsp;

Note that for 5-category ethnicity, group 1 is defined as South Asian and includes Indian, Pakistani, Bangladeshi, and Other Asian, whilst Chinese ethnicity is included in group 3 (Other), as per the 2001 census categories. For the 2011 census, Chinese ethnicity was included in group 1 to make an 'Asian' category.

&nbsp;

\*Conflicts:
* If 5-category ethnicity is 'White' and
  * 16-category ethnicity is 'Indian', 'Pakistani', 'Bangladeshi', 'Other Asian', 'Black Caribbean', 'Black African', 'Other Black', or 'Chinese'
  * QRISK2-category ethnicity is 'Indian', 'Pakistani', 'Bangladeshi', 'Other Asian', 'Black Caribbean', 'Black African', or 'Chinese'
* If 5-category ethnicity is 'South Asian' and
  * 16-category ethnicity is 'White British', 'White Irish', 'Other White', 'White and Black Caribbean', 'White and Black African', 'Black Caribbean', 'Black African', 'Other Black', or 'Chinese'
  * QRISK2-category ethnicity is 'White', 'Black Caribbean', 'Black African', or 'Chinese'
* If 5-category ethnicity is 'Black' and
  * 16-category ethnicity is 'White British', 'White Irish', 'Other White', 'White and Asian', 'Indian', 'Pakistani', 'Bangladeshi', 'Other Asian', or 'Chinese'
  * QRISK2-category ethnicity is 'White', 'Indian', 'Pakistani', 'Bangladeshi', 'Other Asian', or 'Chinese'

&nbsp;

Note that CPRD also provide patient ethnicity (at a fee) based on their own ethnicity algorithm incorporating primary care and HES data. The medcodes which they use do not differ substantially to ours (based on usage in our October 2020 download of patients with diabetes), although their algorithm does differ slightly (paper here: https://pophealthmetrics.biomedcentral.com/articles/10.1186/s12963-023-00302-0).

&nbsp;

### Smoking
* Take most recently recorded smoking category (non-smoker, ex-smoker, active smoker); if both non-smoker and ex-smoker recorded then choose ex-smoker
* If most recent is 'non-smoker' but have previously been recorded as active smoker, then categorise as 'ex-smoker'
* Look at the next most recent date if conflicting categories (ex-smoker and active smoker or non-smoker and active smoker) are recorded on the most recent date
* (NB: different algorithm used for smoking variable used in QRISK2/QDiabetes-Heart Failure: use most recent medcode within last 5 years and QRISK2 category associated with that code (0=non-smoker, 1=ex-smoker, 2=light smoker [1-10 cig/day], 3=moderate smoker [11-19 cig/day], 4=heavy smoker [20+ cig/day]). If both non-smoker and ex-smoker are recorded then choose ex-smoker. However, if medcode has a value associated with it and a valid numunitid (see 'smoking' in biomarker_acceptable_units.txt file in Biomarker directory), assume value is number of cigarettes per day, and use to re-assign those in category 0, 2, 3, and 4 (assume values associated with ex-smoking codes are cig/day that person used to smoke). Ignore values associated with medcode '1780396011' (Cigarette pack/years). If conflicting categories (0 and 2/3/4 or 1 and 2/3/4) recorded on the same day, use minimum.)

&nbsp;

### Alcohol consumption
* Take most recently recorded alcohol consumption level (0 = none, 1 = within limits, 2 = excess, 3 = harmful)
* If ever coded with a level 3 code then categorise as level 3
* If multiple levels coded on most recent date, take the highest level

&nbsp;

### Townsend Deprivation Scores
* 2015 Index of Multiple Deprivation (IMD) deciles were converted to 2011 Townsend Deprivation Scores (TDS) by matching on 2011 Lower Super Output Area (LSOA) and using the median TDS score (to 3 d.p.) of LSOAs with the same IMD decile of the patient
* 2015 IMD decile-2011 LSOA lookup from here: https://www.gov.uk/government/statistics/english-indices-of-deprivation-2015
* 2011 TDS-2011 LSOA lookup from here: https://statistics.ukdataservice.ac.uk/dataset/2011-uk-townsend-deprivation-scores/resource/de580af3-6a9f-4795-b651-449ae16ac2be

&nbsp;

## ICD10 and OPCS4 codelists
* 3 digit ICD10 and OPCS4 codes should be used to find matching 3 digit and 4 digit codes (i.e. those which start with the matching 3 digits)
* ICD10 and OPCS4 codes do not contain lowercase letters, so case sensitive matching is not required
* Episode start date used as date of diagnosis

&nbsp;

## Code list generation process

### Medcodes
In order to generate medcode lists for use in our CPRD Aurum dataset, we developed a flexible pipeline that is adaptable to any condition, biomarker*, or sociodemographic feature (as detailed in the flow diagram below). This process generates Read and SNOMED code lists, as GP systems are switching from using Read codes to SNOMED codes, and maps them to medcodes (CPRD's own coding format). The pipeline uses published code lists from online repositories as inputs where available, as well as term searching and mapping from Read to SNOMED codes. All code lists generated using this pipeline are reviewed by a clinician (with the exception of BMI, height, weight, and blood pressure, where non-clinician review was deemed acceptable).
<img src="https://github.com/Exeter-Diabetes/CPRD-Codelists/blob/main/Images/codelist_generation_pipeline.png?" width="1000">
The final validated Read and SNOMED codelists produced by this pipeline are available in this repository: where available, they are located in each of the subfolders in the [Medcodes](https://github.com/Exeter-Diabetes/CPRD-Codelists/tree/main/Medcodes) folder. Many of our Read codelists include both Read2 and CTV3 codes, however some just include Read2 codes, depending on the source codelists used. Note that Read and SNOMED codelists are csv files and may have hash (#) symbols as suffixes in the 'readcode'/'snomedcode'/'medcodeid' columns to prevent rounding when the csv files are opened in Excel.

*Exception: For most biomarkers (24hr urine protein, ACR, blood albumin, ALT, AST, bilirubin, blood creatinine, urine creatinine, FE1, fasting glucose, haematocrit, haemoglobin, HDL, LDL, PCR, total cholesterol, tryglycerides, TSH, TTG, urine dipstick protein) we have used the NHS Pathology Bounded Code List (PBCL) 2018, a subset of codes used for the majority of test reporting. We term searched the PBCL list for each biomarker (PBCL codes identified are available [here](https://github.com/Exeter-Diabetes/CPRD-Codelists/tree/main/Medcodes/Biomarkers/PBCL%20codelists)) and then mapped the resulting Read/SNOMED codes to medcodes (clinician review not necessary). The PBCL has now been replaced by the PaLM (pathology and laboratory medicine) reference set; in 2024, we checked to see whether including SNOMED codes in the latest PaLM reference set which were not in the 2018 PBCL would significantly improve biomarker capture in a June 2024 CPRD Aurum extract: it did not, and so we are ocntinuing to use the 2018 PBCL. 

In addition, a subset of medcode lists were generated by alternative means: Covid medcodes were provided by CPRD; ethnicity medcodes were provided by LSHTM; hearing loss medcodes were provided by the [GEMINI team](https://github.com/GEMINI-multimorbidity/GEMINI-LTC-code-list-Public/tree/main); care home, bipolar disorder, schizophrenia, pulmonary embolism and deep vein thrombosis medcodes were provided by collaborators; family history, frailty ('frailty_simple'), diabetes, learning disabilities, severe mental illness, surgical pancreatic resection, pancreatic insufficiency, weight loss, polydipsia and various biomarker medcodes (C-peptide, cholesterol:HDL ratio, GAD/IA2/ICA antibodies, HbA1c) were generated by searching for key words/terms in the CPRD Aurum Medical Dictionary for completeness. Read and SNOMED codelists are not available for these conditions/biomarkers.

&nbsp;

### Other
ICD-10, and OPCS-4 code lists do not use this pipeline, but are all put together with input from a clinician. For prodcodes, we generated a list of all generic and brand names for the medication of interest and searched for these in the CPRD Product Dictionary (some lists available in this repository at Prodcode lists/Drug name lists/), and the final codelist was then reviewed by a clinician.
