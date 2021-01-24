# tech-challenge-connecting-people

## R - data preparation manual

### How to prepare datasets for using them in tableau
This is a brief manual on how we prepared the data for our prototype at the example of one dataset. The process can be transferred to all other datasets from DHS. The DHS data was made available in the SPSS file format “.sav” but did not come with meaningful header names and included many variables that are not relevant for the topic of mobility. Moreover, it included some variables that are not consistently filled. To get insight jnto which variables are included and how the raw data is structured, please refer to the *[DHS Standard Recode Manual](https://dhsprogram.com/pubs/pdf/DHSG4/Recode7_DHS_10Sep2018_DHSG4.pdf)*

### Install and load required packages
```
#install.packages("dyplr")
#install.packages("data.table")
#install.packages("foreign")
#install.packages("tidyr")
#install.packages("readxl")
#install.packages('writexl')

library("dplyr")
library("data.table")
library("foreign")
library("tidyr")
library("readxl")
library("writexl")
```

### Load Data into R (Example: Household data 2016)
```
setwd("C:\\Users\\janho\\OneDrive\\TechChallenge_Connection People\\Prototype\\Tableau Map")
data_sav <- read.spss("ETHR71FL.SAV")

HR_dt_1 <- as.data.table(data_sav)
```

### Rename the variables
#### 1. First load the excel file that contains the mapping from variable code to variable name
You can find the file “Mobility Indicators DHS_universal.xlsx” in the file repository
```
indicators_dt <-read_excel("C:\\Users\\janho\\OneDrive\\TechChallenge_Connection People\\Prototype\\Tableau Map\\Mobility Indicators DHS_universal.xlsx", sheet = "Mobility")
setDT(indicators_dt)
class(indicators_dt)
View(indicators_dt)
```

#### 2. Extract the rows that contain the relevant identifier. In this case: “Household_hh” from “indicators_dt”.
```
household_dt <- indicators_dt[Dataset == "Household_hh"]
Individual_fem_dt <- indicators_dt[Dataset == "Individual_fem"]
Men_m_dt <- indicators_dt[Dataset == "Men_m"] 

View(household_dt)
```

#### 3. Extract the variable codes and short names from “household_dt” and remove NAs
```
codes_names_dt_NA <- household_dt[, c("Variable code", "Short Name")]
View(codes_names_dt_NA)
codes_names_dt <- codes_names_dt_NA[!is.na(codes_names_dt_NA$"Variable code"), ]
View(codes_names_dt)
```

#### 4. Get vectors of variable code and short name
```
codes <- codes_names_dt$`Variable code`
names <- codes_names_dt$`Short Name`
```
#### 5. Change column names
```
renamed_HR_dt <- setnames(HR_dt_1, codes, names, skip_absent=TRUE)
#manual change:
setnames(renamed_HR_dt, c("HV001", "HV002"), c("DHSCLUST", "Household Number"))
```
#### 6. transform into data table
```
renamed_dt2 <- as.data.table(renamed_HR_dt)
```
### Cleaning the data: get rid of variables unrelated to mobility (as specified in the mapping file)
```
HR_end_dt <- renamed_HR_dt %>% select(-contains("HV"))
HR_end_dt_nonum <- HR_end_dt %>% select (-ends_with("1"), 
                                         -ends_with("2"),
                                         -ends_with("3"),
                                         -ends_with("4"),
                                         -ends_with("5"),
                                         -ends_with("6"),
                                         -ends_with("7"),
                                         -ends_with("8"),
                                         -ends_with("9"),
                                         -ends_with("0"))                          
clean_HR_2016 <-HR_end_dt_nonum %>% select(-starts_with("SH"), -starts_with("IDX"))
View(clean_HR_2016)


# Check for NAs and if necessary, delete the variables with many NAs in the previous step
na_count <-sapply(clean_HR_2016, function(y) sum(length(which(is.na(y)))))
na_count <- data.frame(na_count)
na_count
```

### Export the tansformed datafile into .csv and .xlsx
```
write.csv(clean_HR_2016,"C:\\Users\\janho\\OneDrive\\TechChallenge_Connection People\\Prototype\\Tableau Map\\clean datasets\\clean_HR_2016.csv")

write_xlsx(clean_HR_2016, "C:\\Users\\janho\\OneDrive\\TechChallenge_Connection People\\Prototype\\Tableau Map\\clean datasets\\clean_HR_2016.xlsx")
```
