# -Patient-Encounter-Cost-and-Risk-Analysis-in-Healthcare-Systems

1. Patient Encounter Cost and Risk Analysis in Healthcare Systems.

   This project focuses on analyzing healthcare cost data to uncover key insights related to encounter types, patient utilization, procedure trends, payer coverage gaps, and regional cost disparities. The dashboard     provides a comprehensive, data-driven view to support strategic healthcare planning and resource optimization.

2. Key business question.

   Which encounter types contribute most to overall healthcare costs?

   Who are the high-utilization patients, and what is driving their costs?

   Which portions of care are not covered by payers, and what are the common denial reasons?

   How do procedure costs change over time, and are there any cost spikes or inefficiencies?

   Which regions incur the highest healthcare expenses, and what patterns can be observed geographically?

3. Dataset.

   Encounters.csv,

   Patients.csv,

   Payers.csv,

   Organizations.csv,

   Procedures.csv .

4. Project Structure.

   Healthcare_Cost_Analysis/

   
   ├── README.md
   
   ├── Project_Report.docx / .pdf
   
   ├── Data_Dictionary.xlsx
   
   ├── database_schema.sql
   
   ├── Healthcare_Cost_Dashboard.pbix  (Power BI) / .twbx (Tableau)


   
   ├── /data/
   
   │   └── raw_data.csv
   
   │   └── cleaned_data.csv

   
   ├── /sql/
   
   │  ├── 01_data_loading.sql
   
   │  ├── 02_data_cleaning.sql
   
   │  └── 03_analysis_queries.sql

   
   ├── /notebooks/ (Optional for Python/SQL notebooks)
   
   │  └── data_exploration.ipynb


   ├── /visualizations/
   
   │  ├── encounter_cost_distribution.png
   
   │  ├── high_cost_patients.png
   
   │  ├── uncovered_costs_analysis.png
   
   │  ├── procedure_cost_trends.png
   
   │  └── regional_cost_analysis.png


   
   └── /documents/
   
    ├── SQL_Analysis_Results.xlsx
   
    └── PowerBI_Storyboards.pptx


   
6. Tools requuired.

    SQL Server Management Studio - Main data cleaning or the sql querry structure are made on the server studio . 

    Power BI Desktop -  Main Data Visualization platform is used for report creation.

    Power Querry - Data transformation  and cleaning layer for reshaping for reshaping and preparing the data.

    DAX (Data Analysis Expression)  - Used for calculated measures, dynamic visuals, and conditional logic.

    Data Modeling - Releationship established among tables (encounters,  organizations, patients, payers, procedures) to enable cross-filtering and aggregation

    File Format - .pbix for development and .png for dassboard previews.



7. 1.  Data Exploration & Preprocessing

         SQL Logic :

                   SELECT
       
                      SUM(CASE WHEN TOTAL_CLAIM_COST IS NULL THEN 1 ELSE 0 END) AS Missing_TotalClaimCost,

                      SUM(CASE WHEN PAYER_COVERAGE IS NULL THEN 1 ELSE 0 END) AS Missing_PayerCoverage,

                      SUM(CASE WHEN REASONCODE IS NULL THEN 1 ELSE 0 END) AS Missing_ReasonCode,

                      SUM(CASE WHEN PATIENT IS NULL THEN 1 ELSE 0 END) AS Missing_PatientID

                  FROM Encounters;


                  SELECT ENCOUNTERCLASS, COUNT(*) AS DuplicateCount

                  FROM encounters

                  GROUP BY ENCOUNTERCLASS

                  HAVING COUNT(*) > 1;


                 WITH DuplicatesCTE AS (
       
                      SELECT *,
       
                          ROW_NUMBER() OVER (PARTITION BY ENCOUNTERCLASS ORDER BY ENCOUNTERCLASS) AS rn
   
                       FROM encounters
       
                     )
       
                       DELETE FROM DuplicatesCTE WHERE rn > 1;


                       DELETE FROM encounters WHERE id IS NULL OR PATIENT IS NULL;
      



   3. (a) Evaluating Financial Risk by Encounter Outcome

          SQL Logic :

              SELECT
      
              REASONCODE,
      
              SUM(TOTAL_CLAIM_COST - PAYER_COVERAGE) AS TotalUncoveredCost,
      
              COUNT(*) AS EncounterCount
      
              FROM encounters
      
              WHERE TOTAL_CLAIM_COST IS NOT NULL AND PAYER_COVERAGE IS NOT NULL

              GROUP BY REASONCODE
      
              ORDER BY TotalUncoveredCost DESC;

            

      (b) Identifying Patients with Frequent High-Cost Encounters

          SQL Logic :

          
                WITH HighCostEncounterCount AS (

                      SELECT
          

                            PATIENT,
             
                            YEAR(START) AS EncounterYear,
             
                            COUNT(*) AS HighCostEncounterCount
             
                      FROM encounters
          
                      WHERE TOTAL_CLAIM_COST > 10000
          
                      GROUP BY PATIENT, YEAR(START)
          
                 )
         
                     SELECT 
    
                          PATIENT,
           
                          ENCOUNTERCLASS,
           
                          TOTAL_CLAIM_COST
           
                     FROM encounters
         
                     WHERE TOTAL_CLAIM_COST > 3
         
                     ORDER BY TOTAL_CLAIM_COST DESC, PATIENT;
    


      (c) Identifying Risk Factors Based on Demographics and Diagnosis Codes
    
          SQL Logic :

          
                  WITH Top3Diagnosis AS (
                  
                                  SELECT TOP 3 
                                  
                                      CODE, 
                             
                                   COUNT(*) AS DiagnosisCount
                                   
                                  FROM encounters
                                  
                                  WHERE CODE IS NOT NULL
                                  
                                  GROUP BY CODE
                                  
                                  ORDER BY DiagnosisCount DESC

                              )

                           SELECT
                           
                               t.CODE,
                               
                               e.PATIENT,
                               
                               e.ENCOUNTERCLASS,
                               
                               e.REASONCODE,
                               
                               e.TOTAL_CLAIM_COST
                               
                           FROM encounters e
                           
                           JOIN procedures t
                           
                           ON e.CODE = t.CODE
                           
                           ORDER BY t.CODE, e.PATIENT;
    


      (d) Assessing Payer Contributions for Different Procedure Types
    
          SQL Logic :

                   SELECT 
                   
                         CODE,
                         
                         COUNT(*) AS ProcedureCount,
                         
                         SUM(TOTAL_CLAIM_COST) AS TotalClaimCost,
                         
                         SUM(PAYER_COVERAGE) AS TotalPayerContribution,
                         
                         SUM(PAYER_COVERAGE) * 1.0 / NULLIF(SUM(TOTAL_CLAIM_COST), 0) AS PayerContributionRatio
                         
                         FROM encounters
                         
                         WHERE CODE IS NOT NULL
                         
                         GROUP BY CODE
                         
                         ORDER BY PayerContributionRatio ASC;


      (e) Identifying Patients with Multiple Procedures Across Encounters

          SQL Logic :

                
                     WITH PatientDiagnosisProcedures AS (
                     
                                              SELECT
                                              
                                                  PATIENT,
                                                  
                                                  CODE,
                                                  
                                                  COUNT(DISTINCT Id) AS DistinctEncounters,
                                                  
                                                  COUNT(DISTINCT CODE) AS DistinctProcedures
                                                  
                                             FROM encounters
                                             
                                             WHERE REASONCODE IS NOT NULL AND CODE IS NOT NULL
                                             
                                             GROUP BY PATIENT, CODE
                                             
                              )
                                            SELECT
                                            
                                               PATIENT,
                                               
                                               REASONCODE,
                                               
                                               ORGANIZATION,
                                               
                                               PAYER
                                               
    
                                           FROM encounters
                                           
                                           WHERE REASONCODE > 1 AND CODE > 1
                                           
                                           ORDER BY PATIENT, CODE;

    

      (f) Analyzing Patient Encounter Duration

          SQL Logic :

          
                   SELECT 
                   
                      ORGANIZATION,
                      
                      Id,
                      
                      PATIENT,
                      
                      START,
                      
                      STOP,
                      
                      DATEDIFF(HOUR, START, STOP) AS EncounterDurationHours
                      
                  FROM Encounters
                  
                  WHERE START IS NOT NULL 
                  
                 AND STOP IS NOT NULL
                 
                 AND DATEDIFF(HOUR, START, STOP) > 24
                 
                 ORDER BY ORGANIZATION, EncounterDurationHours DESC;

    

   
   
8. Business Impacts and Insights.


    1. Encounter Cost Distribution
       
       Insight:
       
       Breaks down healthcare costs by EncounterClass (e.g..).

       Business Impact:

       Identifies which types of encounters drive the highest costs.

       Helps healthcare administrators optimize budgeting and staffing based on service demand.

       Supports cost-reduction strategies in high-expense care settings (e.g., ).

   2. High-Cost Patient Identification
      
      Insight:
      
      Highlights patients with frequent or expensive healthcare encounters (e.g., more than 3 visits/year costing over $10,000).

      Business Impact:

      Enables early intervention and case management for high-risk patients.

      Reduces preventable hospital visits through targeted chronic care programs.

      Improves patient outcomes while lowering long-term costs.

   4. Uncovered Costs Analysis
      
      Insight:

      Examines gaps between total claim cost and payer contributions, categorized by PayerType and ReasonCode.

      Business Impact:

      Reveals systemic issues in insurance coverage or billing processes.

      Helps reduce revenue loss by addressing frequent denial reasons.

      Supports negotiations with insurers by providing denial trends and justification data.

 4. Procedure Cost Trends
    
    Insight:
    
    Shows how procedure-related costs vary over time (monthly/annually), helping to spot patterns or cost inflation.

    Business Impact:

    Supports evaluation of cost-effectiveness for procedures and treatments.

    Helps in detecting overuse, underuse, or cost spikes in specific services.

    Informs procurement and pricing decisions for medical technologies or supplies.

5. Geographical Analysis
   
   Insight:
   
   Identifies regions or locations with disproportionately high healthcare costs.

   Business Impact:

   Assists in targeting regional health programs or outreach efforts.

   Informs decisions on resource allocation, staffing, or facility placement.

   Helps public health agencies prioritize areas for cost control or health education.

   
   
10. Screenshots / Demos.

    Show what the dashboard looks like : -

    Example : [Dashboard preview] : (https://github.com/Pawan-vishwa/-Patient-Encounter-Cost-and-Risk-Analysis-in-Healthcare-

              Systems/blob/main/Patient%20Encounter%20Cost%20and%20Risk%20Analysis%20in%20Healthcare%20Systems.png )

 <img width="1167" height="756" alt="Patient Encounter Cost and Risk Analysis in Healthcare Systems" src="https://github.com/user-attachments/assets/00903927-7722-4bea-b409-02fca3091001" />


    
