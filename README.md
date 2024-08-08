# behavior-monitoring
Repo for project ***Behavior Monitoring***

Aim is to predict whether a patient has Mild Cognitive Impairment (MCI) or not based on various demographic, health, and lifestyle factors.

The dataset used in this analysis contains information on patients, including their age, education level, cognitive test scores, gait measurements, and other relevant features including 'watch sensor data (steps, sleep duration)'

This repo has code, input data files, output results obtained.

1. **Reproducing Home mobility and Frequency stability paper**  
Run the democlean file first for preprocessing the data, then ProcessSensor file to reproduce results.  
    * Code: DemoClean.py, ProcessSensor.py
    
    - *DemoClean.py:*
        * *Input data files:* 
            - Single_Resident_Homes_CART.csv: contains subids along with study and home id information

            - CART_long_demo.csv: contains 707 features with details such as age, gender, education, race, occupation, lifestyle survey (smoking, alcohol intake, medication) and NACC (National Alzheimer's Coordinating Center) variables such as health conditions, disorders, recall and language test scores, etc., 

            - 20201201_4subid.csv: contains demographic data for 4 participants from OHSU, to be merged with `CART_long_demo.csv`

            - mars.csv: demographic data for participants from RUSH cohort. These individuals reside in Chicago.

            - gait speed data 04-2021.csv: gait speed data for RUSH cohort

            - oll_g.csv: needed for homeid information for OLL participants

            - Wu OLL VSTS 091020.csv: contains demographic data for OLL study such as subid, gender, education, age and living information, etc.

            - N348 SDSQ 091620.csv: sleep quality survey data for OLL

            - N348 GAIT SPEED CIRS 09222020.csv: gait data for OLL

            - Cart_annual_v1.csv: contains sleep quality survey data for OHSU, VA, RUSH, and MIAMI 

            - CART_number_rooms.csv: contains number of rooms per homeid

            - rush_additional.csv: contains missing information for some RUSH IDs

            - rush_final_20210708.csv: same structure as mars.csv, contains multiple records for each participant instead of 1 per subid. Does not any new information compared to mars.csv


        * Results: 
            - CART_OLL_demo_sleep.csv: combined demographic, gait and sleep quality data for OHSU, VA, RUSH and OLL studies

    
    - ProcessSensor.py:
        * *Input Data files:*  
            - Single_Resident_Homes_CART+OLL.csv: Contains study, homeid, and subid

            - 20200914_huf.csv: weekly survey questionnaire for CART (OHSU, VA, RUSH)

            - 20200910_huf_oll.csv: weekly survey questionnaire for OLL

            - Dur_[homeid].csv: PIR room sensor data for each homeid, denoting the duration spent in each area of subject's home, recorded each day

        
        * Results:
            - IVIS_demo_042321.csv: CART_OLL_demo_sleep.csv merged with IV, IS and outdoor time calculations from subject's room-to-room transitions (trips)

            - before_042321.csv: IVIS_demo data after manual updates of participant's demographic and MCI status 

            - exclude_17.csv: demographic + survey data. Final output of DemoClean & ProcessSensor scripts

            - IS_distribution.png: histogram of IS distribution with mean and SD
            - IS_MCI.png: Violin plot of Normal vs MCI pariticpants 
            - trip_distribution.png: histogram of trips with mean and SD




2. **Merged weekly clinical data with watch sensor step and sleep data after converting daily sensor to weekly.**
   
    * *Code:* merge_clinical_watch.ipynb
   
   **2a: Consolidate clinical data with sleep and step data**

    * *Input data files:* 
        - exclude_17.csv: demographic + survey data. Final output of DemoClean & ProcessSensor scripts

        - All csvs of sleep and step (OHSU, RUSH & VA-CART)

    * *Results:* 
        - Intermediate files  # Remove all intermediates here
            - watch_sleep_weekly_data.csv: weekly sleep data after 3-sigma rule is applied for anomalous sleep duration

            - watch_step_weekly_data.csv: weekly step data after filtering for people with disabilities i.e., less than 97 steps per day

            - watch_sleep_data.csv: daily sleep data obtained after merging sleep data for OHSU, VA and RUSH

            - watch_step_data.csv: daily sleep data obtained after merging sleep data for OHSU, VA and RUSH
        
        - weekly_step_excl_17.csv (Clinical + Step data): Sample size of 80; weekly step data enriched with demographic & survey information
        - weekly_step_sleep_excl_17.csv (Clinical + Step & Sleep data): Sample size of 80;weekly step & sleep data data enriched with demographic & survey information

   **2b:Extracting and adding additional features using Watch_Raw_Data**

    Additional features added: 
    - time_to_bed
    - time_to_wakeup
    - tracking step count based on time intervals (morning, afternoon, evening, midnight)
    - for step data after converting raw data to hourly:
        - IV (Interdaily Variability)
        - IS (Interdaily Stability)
        - Acro (Average peak of steps over an interval of 10 days)
        - Nadir (Average lowest # of steps over an interval of 5 days)
        - Amp (Difference between highest and lowest # of steps on average i.e., Acro - Nadir)
        - Trip (Weekly average number of steps calculated using watch raw data)


    * *Input data files:* Watch_Raw_Data files
    
    * *Results:* 
        - Intermediate files:
            - step_counts_intervals.csv: After tracking steps wrt time intervals (replaced by chunked step counts)
            - hourly_data_watch.csv: Watch_Raw_Data grouped by subid, date, hour to log # of steps
            - hourly_data_watch_2.csv: Watch_Raw_Data grouped by subid, date, hour to log # of steps, used as source for hourly data
            - chunked_step_counts.csv: Also for tracking steps wrt time intervals but updated logic
            - calculated_features.csv: IV, IS, Acro, Nadir, Amp for steps computed using hourly data, computed on weekly basis
            - merged_addnl_features.csv: demographic, survey, weekly step & sleep data, along with IV, IS, acro, nadir and chunked steps
            

3. **Obtain 3 month baseline period**

    <u>Approach 3a:</u> Tried using use actual baseline dates, there is subid mismatch with actual baseline dates as some patients might not use sensor, etc. (went ahead with Approach 3b)
    * *Input data files:* weekly_step_sleep_excl_17.csv
    * *Code:* ~~obtain_first3month_baseline.ipynb~~
    * *Results:* record_counts.csv  
    
   <u>Approach 3b:</u> Used the first report date in clinical data and claculated three month period for each subid
    * *Input data files:* weekly_step_sleep_excl_17.csv, clin_fin_red.xlsx
    * *Code:* merge_clinical_watch.ipynb       **Same file as in 2a**
    * *Results:* first_3_month_period.csv: final output used in classifier, filtered for first 3 months of data for each participant




4. **Predicted Baseline Cognitive Status**  
    Two datasets used: one with 639 records, one with 82 records with one record per each subid  

    Models built: Decision tree, Logistic Regression, SVM, Random forest classifier with 100 iterations of training and testing with different split each time.  

    <u>Dataset 1:</u> 639 records  

    * **Code:** Predict_MCI_639_records.ipynb

    * **Input data files:** first_3_month_period.csv

    * **Results:** 
        - Intermediate files
            - imputed_first3months.csv: final output used in classifier, filtered for first 3 months of data for each participant, missing data has been filled

   <U>Dataset 2:</U> 82 records  

    * **Code:** Predict_MCI_82_records.ipynb

    * **Input data files:** first_3_month_period.csv

    * **Results:** 
        - Intermediate files
            - imputed_first3months.csv: final output used in classifier, filtered for first 3 months of data for each participant, missing data has been filled
            - averaged_data.csv: 1 record per participant, generated by aggregating data for each subid
    
   <u>**Data Analysis.ipynb:**</u> Analyzing features and their distributions for MCI and non-MCI patients





5. **Feature Selection**  
    Used Random Forest Classifier's feature_importances_ for the 82 records data to get top 5 features being selected with their frequency in 100 iterations
    * *Input data files:* averaged_data.csv
    * *Code:* Feature selection.ipynb
    * *Results:* Plots

**Miscellaneous:**  
Gaussian_mixture_dist.ipynb - To check the distribution of data

References: 
- [MCI Vs Non-MCI Analysis and predictions](https://docs.google.com/presentation/d/1kpHj0ah_ZIl4KffgTYmRLf4ENJ035o94jjoxPYyoVo8/edit#slide=id.g27910489dc6_0_0)
- [Predicting MCI - step by step](https://docs.google.com/presentation/d/1UkbS2jl-HcUKHzQaCstj-UHBXmdA5YYQWOVFNQjRJAk/edit#slide=id.g237a9984310_0_74)
- [CART Data Analysis](https://docs.google.com/presentation/d/1alFZxRiF812gRXS6v0nNOKuZYWmf4n0V-sJNC4Y_CHU/edit#slide=id.p)
- [Research papers - Analysis and Understanding](https://michiganstate-my.sharepoint.com/:p:/g/personal/kondabat_msu_edu/EX68BiCAwCdMhzFWnpwVTKoBJde-OwJP6rGX95DkeWnP1Q?e=nMfKnc)



## Unused files
- 82_records_data.csv
