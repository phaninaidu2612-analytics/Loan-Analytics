# Loan Default Analysis Dashboard (Source : Dataflow)

### Dashboard Link : 
https://app.powerbi.com/links/X2KOdQBg3U?ctid=06cd4ed5-3b7f-47b0-8ef6-29dee1c5060b&pbi_source=linkShare&bookmarkGuid=b0a4724e-fb75-4d31-bfc9-b1850e1de915


## Dashboard Preview

![Page 1](https://github.com/user-attachments/assets/932209af-e2ed-4f70-8050-d907911d6abc)
![Page 2](https://github.com/user-attachments/assets/efeba583-25af-4cf9-9c36-5a7f14ad46e4)
![Page 3](https://github.com/user-attachments/assets/37495d7b-573b-4156-ac94-9f55f8fa7326)

---

## Problem Statement

This dashboard helps financial institutions understand borrower behavior and identify risk patterns associated with loan defaults. It enables analysis of how different factors such as income, employment type, credit score, and demographics influence loan repayment.

Through this dashboard, lenders can:
- Identify high-risk borrower segments
- Analyze default trends over time
- Understand how credit score and income affect loan amounts
- Evaluate financial stability indicators such as DTI ratio and employment

Since default rates vary significantly across employment types and years, institutions can use this analysis to improve risk assessment and lending strategies.

---

## Dataset Description

The dataset contains information about borrowers, including financial details, loan characteristics, and repayment behavior.

### Key Columns:

- LoanID – Unique identifier for each loan  
- Age – Borrower's age  
- Income – Annual income  
- LoanAmount – Loan amount  
- CreditScore – Creditworthiness score (300–850)  
- MonthsEmployed – Employment duration  
- NumCreditLines – Active credit lines  
- InterestRate – Loan interest rate  
- LoanTerm – Loan duration  
- DTIRatio – Debt-to-Income ratio  
- Education – Education level  
- EmploymentType – Type of employment  
- MaritalStatus – Marital status  
- HasMortgage – Mortgage indicator  
- HasDependents – Dependents indicator  
- LoanPurpose – Reason for loan  
- HasCoSigner – Co-signer indicator  
- Default – Whether loan was defaulted  
- Loan Date – Loan issuance date  

---

## Steps Followed



### Data Preparation
- Create a work space in PowerBI service.At the top there is a download icon->Data gateway ->we installed a data gateway standard edition 
- In our workspace , there is an option called new item->a box appears and in that searh for data
  and choose Dataflow gen1.Then choose SQL server database
- Enter the sql server windows credentials or sql server database login credentials.
- Loaded dataset into Power BI Desktop (Searching Dataflows in get data and choose your sql server database and import the data)
- Used Power Query for data profiling
- Checked column distribution, quality, and profile
- Cleaned and validated dataset


---

### Data Transformation

Year = YEAR('Loan_default'[Loan_Date_DD_MM_YYYY])

Age group = 
 IF('Loan_default'[Age] <= 19 , "Teen" , 
    IF('Loan_default'[Age] <=39 , "Adult" ,
        IF('Loan_default'[Age] <= 59 , "Middle Age Adults",
        "Senior citizen")))

Snap of the New created column: ![age_group_column](https://github.com/user-attachments/assets/cec19ba1-8f40-4893-8c7b-b442d0630847)

Credit score bins = 
 IF('Loan_default'[CreditScore] <= 400 , "Very Low" , 
    IF('Loan_default'[CreditScore] <= 450 , "Low" , 
        IF('Loan_default'[CreditScore] <= 650 , "Medium", "High")))

Snap of the New created column: ![credit_bin](https://github.com/user-attachments/assets/d35f17ea-cd2c-47b4-8176-7fead03c7f15)

Income bracket = 
 SWITCH(TRUE(), 
    'Loan_default'[Income] < 30000 , "Low Income",
    'Loan_default'[Income] >=30000 && 'Loan_default'[Income] < 60000 , "Medium Income" , 
    'Loan_default'[Income] >= 60000 , "High Income")

Snap of the New created column: ![income_bracket](https://github.com/user-attachments/assets/4b67c763-2627-499e-a1c5-8aa8e25d18da)

## Measures (DAX)

YOY Default Loans change = 
DIVIDE(
    CALCULATE(COUNTROWS(FILTER('Loan_default','Loan_default'[Default]=TRUE())),
    'Loan_default'[Year]=YEAR(MAX('Loan_default'[Loan_Date_DD_MM_YYYY]))) -
    CALCULATE(COUNTROWS(FILTER('Loan_default','Loan_default'[Default]=TRUE())),
    'Loan_default'[Year]=YEAR(MAX('Loan_default'[Loan_Date_DD_MM_YYYY]))-1),
    CALCULATE(COUNTROWS(FILTER('Loan_default','Loan_default'[Default]=TRUE())),
    'Loan_default'[Year]=YEAR(MAX('Loan_default'[Loan_Date_DD_MM_YYYY]))-1),0) * 100

Snap of the DAX : ![yoy_default_loan_change](https://github.com/user-attachments/assets/897c1a70-2670-4dfe-a060-2b6bdd5efb1f)



YOY Loan Amount Change = 
DIVIDE(
    CALCULATE(SUM('Loan_default'[LoanAmount]),
    'Loan_default'[Year]=YEAR(MAX('Loan_default'[Loan_Date_DD_MM_YYYY]))) -
    CALCULATE(SUM('Loan_default'[LoanAmount]),
    'Loan_default'[Year]=YEAR(MAX('Loan_default'[Loan_Date_DD_MM_YYYY]))-1),
    CALCULATE(SUM('Loan_default'[LoanAmount]),
    'Loan_default'[Year]=YEAR(MAX('Loan_default'[Loan_Date_DD_MM_YYYY]))-1),0) * 100

    

YTD Loan Amount = 
CALCULATE(SUM('Loan_default'[LoanAmount]),
DATESYTD('Loan_default'[Loan_Date_DD_MM_YYYY].[Date]),
ALLEXCEPT('Loan_default',Loan_default[Credit Score Bins],Loan_default[MaritalStatus]))

Snap of DAX : ![ytd](https://github.com/user-attachments/assets/1a3d176f-5348-4772-a7e4-a31ff1a54ad1)



Average Loan Amt (High Credit) = 
AVERAGEX(FILTER('Loan_default','Loan_default'[Credit Score Bins]="High"'),'Loan_default'[LoanAmount])

Loans by Education type = 
COUNTROWS(FILTER('Loan_default',NOT(ISBLANK('Loan_default'[LoanID]))))

Median by Credit score bins = 
MEDIANX('Loan_default','Loan_default'[LoanAmount])

Total Loan (Credit Bins) = 
CALCULATE(SUM('Loan_default'[LoanAmount]),
'Loan_default'[Age Groups]="Adults",
ALLEXCEPT('Loan_default','Loan_default'[Age],'Loan_default'[Age Groups],'Loan_default'[CreditScore],'Loan_default'[Credit Score Bins]))

Total Loan (Middle Age Adults) = 
SUMX(FILTER('Loan_default','Loan_default'[Age Groups]="Middle Age Adults"'),'Loan_default'[LoanAmount])

Average Income by Employment type = 
CALCULATE(AVERAGE('Loan_default'[Income]),
ALLEXCEPT('Loan_default','Loan_default'[EmploymentType]))

Average Loan by Age Group = 
AVERAGEX(VALUES('Loan_default'[Age Groups]),
AVERAGE('Loan_default'[LoanAmount]))

Default Rate by Employment type = 
VAR totalrecords = COUNTROWS(ALL('Loan_default'))
VAR DefaultCases = COUNTROWS(FILTER('Loan_default','Loan_default'[Default]=TRUE()))
RETURN
CALCULATE(DIVIDE(DefaultCases,totalrecords),
ALLEXCEPT('Loan_default','Loan_default'[EmploymentType])) *100

Snap of the DAX : ![default_emp_type](https://github.com/user-attachments/assets/eb2fe5b0-4491-4dd5-9eb7-50a7a748cc52)

Default Rate by Year = 
VAR totalloans = CALCULATE(COUNTROWS('Loan_default'),
ALLEXCEPT('Loan_default',Loan_default[Year]))
VAR default = CALCULATE(COUNTROWS(FILTER('Loan_default','Loan_default'[Default]=TRUE())),
ALLEXCEPT('Loan_default',Loan_default[Year]))
RETURN
DIVIDE(default,totalloans)*100

Loan Amount by Purpose = 
SUMX(FILTER('Loan_default',NOT(ISBLANK('Loan_default'[LoanAmount]))),
'Loan_default'[LoanAmount])

---
##Submitting the report to PowerBI:

![publish](https://github.com/user-attachments/assets/cdeee4c7-68fe-4ad9-87c3-a64894482dd6)

##Snapshot of Dashboard (Power BI Service):

![powerbi_service](https://github.com/user-attachments/assets/2c25e266-4d65-4ce2-8494-5758b9ec26ae)

## Scheduled Refresh

- Implemented scheduled refresh in Power BI Service to automatically update the dataset with the latest loan records  
- Enabled continuous monitoring of key metrics such as default rates, loan distribution, and income segmentation  
- Reduced manual effort by automating data updates, ensuring timely and accurate decision-making  

![sr](https://github.com/user-attachments/assets/e0aed5c7-4fe2-40c7-867a-b735e9ad2a5f)

## Features

- Multi-page dashboard
- Risk analysis using default rates
- Time-based analysis (YOY, YTD)
- Demographic segmentation
- Financial profiling



## Insights

A single page report was created on Power BI Desktop & it was then published to Power BI Service.

Following inferences can be drawn from the dashboard

Default rate by employment type:
- Unemployed borrowers exhibit the highest default rate (~3.39%), indicating a significantly higher financial risk compared to other employment types  
  Full-time employed borrowers show the lowest default rate (~2.36%), suggesting greater financial stability and repayment capability  
  Default risk decreases as employment stability increases, highlighting employment type as a key factor in credit risk assessment.

![snap_default](https://github.com/user-attachments/assets/b6ee647f-a1eb-499b-a10f-2987731401d3)

High credit score customers show stable patterns:
- High credit score borrowers exhibit consistent loan amounts (~123K–128K) across different age groups and marital statuses, indicating stable borrowing behavior  
  This consistency reflects stronger financial reliability, making high credit individuals lower-risk segments for lending institutions  

![snap_highcredit](https://github.com/user-attachments/assets/811318c3-ecfe-4c9c-9c74-912f7757ce76)

Age Influences Loan Amount:
- Adults (20–39 age group) show the highest average loan amount (~127.9K), indicating strong borrowing activity during early career stages  
  Loan amounts show a slight decline with age, with teenagers having the lowest borrowing levels due to limited financial independence  
  This trend suggests that borrowing demand is highest during active earning and growth phases of life  

![age_influence](https://github.com/user-attachments/assets/36f3cd00-e4a3-4793-bdd9-566bcb94e7e7)
   

Income influences Loan Amount:
- Loan distribution increases significantly with income level, with high-income borrowers contributing the majority (~21.73bn), while low-income groups account for a relatively smaller share (~3.63bn)  
  This suggests that income level plays a key role in determining borrowing capacity and loan size  

![income_influence](https://github.com/user-attachments/assets/2a93c668-e7d9-46f7-ac34-7ce5a3087326)

-------



## Business Impact

- Helps identify high-risk borrower segments  
- Supports better credit approval decisions  
- Enables data-driven risk management  
- Reduces potential loan defaults  

--

## Tools Used

- Power BI
- DAX
- Data Modeling
