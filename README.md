---
title: "Investigating COVID-19 Virus Trends (R Fundamentals)"
output: html_notebook
---
# Introduction

COVID-19 is one of the greatest viruses to exist in the 21st century, everyday people battle against it to mitigate its damage. Although data is being compiled every minute of the day, these data sets are largely incomplete given that some cases will never be reported. The current goal is to test as many people as possible so that we can allocated scarce resources to patients who need it the most. Where we currently fall short is not focusing on creating a comprehensive data set for epidemiolgists to study. In anycase, its entirely possible for countries who are testing to have a larger infected population than are being reported. For instance country's that provide minimal testing can run the risk of not knowing the true spread of the virus. On the other hand, countries that show an increasing number of cases might indicated that testing is catching the true numbers of reported COVID19 cases.

Although the true number of COVID-19 will never be known, we do our best to understand its spread by using our testing data. What we do know is that test are being reserved for those who are showing symptoms of the virus. Given that the virus effects people differently, those who are asymptomatic may not get tetsed if they are not being impacted dramatically. Meaning those who are getting tested are people with medium levels of health issues associated with COVID-19 and not those who are being infected but show little sign of health problems. This may also describe why we see the disparity of positive cases between the age demographic in many datasets. In summary, tracking this virus is very complicated to say the least, there are many variables to consider when trying to understand the spread of the virus in your country.

## Data Set

The data used for this analysis was from [COVID19 Worldwide Testing Data](https://www.kaggle.com/lin0li/covid19testing) on kaggle and was prepared by [dataquest.io](https://www.dataquest.io/)

### Project Objective

The objective of this project is to demonstrate R data structure fundamentals, which include vectors, matrices, lists, and dataframes.


### Question
Question of interest: Which Countries have had the highest number of positive cases against the number of tests?


```{r}
#loading data and packages
library(tibble)
library(readr)
library(tidyr)
library(purrr)
library(dplyr)
covid_df <- read_csv('covid19.csv')
```

### First Look At The Dataframe

The dim function allows us to retrieve the dimensions of the dataframe, which shows us the total number of rows followed by the total number of columns.
```{r}
#determine dimensions of dataframe
dim(covid_df)
```

Here we are saving the column names to a vector for later use.
```{r}
#save column names to a vector for later
vector_cols <- colnames(covid_df)
vector_cols
```

Head fucntion allows us to peer into the dataframe and retrieve the number of rows 1 - n
```{r}
#check first few rows of data
head(covid_df, 3)
```

the glimpse function in the tibble package combines R functions of dim and head to give you a better look at the dataframe. 
```{r}
#display a summary of the dataframe
glimpse(covid_df)
```

### Isolating Rows

we can see that the Province_state mixes data from different levels: country  and state/province. Since we only want the full country, we can filter out all points that are not designated as "All States". After filtering we dont need to keep the Province_State column since all the data will be the same.

```{r}
#filter Province_State column and removing Province_state column
covid_df_all_states <- covid_df %>%
  filter(Province_State == "All States") %>%
  select(-Province_State)
```


```{r}
#check our filter
head(covid_df_all_states,3)
```


After looking at the data set, we notice that there are columns that hold daily information and those that provide cumulative information. To avoid the mistake of comparing cumulative data to daily data we will work with daily only.
```{r}
#select the data that keeps daily information
covid_df_all_states_daily <- covid_df_all_states %>%
  select(Date, Country_Region, active, hospitalizedCurr, daily_tested, daily_positive)
```


### Answering the question

```{r}
#find the top ten tested countries
covid_df_all_states_daily_sum <- covid_df_all_states_daily %>%
  group_by(Country_Region) %>%
  summarise(tested = sum(daily_tested),
            positive = sum(daily_positive),
            active = sum(active),
            hospitalized = sum(hospitalizedCurr)) %>%
  arrange(desc(tested))
covid_top_10 <- head(covid_df_all_states_daily_sum,10)
covid_top_10
```




```{r}
#create vector for each column in the covid_top_ten dataframe
countries <- covid_top_10$Country_Region
tested_cases <- covid_top_10$tested
positive_cases <- covid_top_10$positive
active_cases <- covid_top_10$active
hospitalized_cases <- covid_top_10$hospitalized
```


```{r}
#Add name sto the new vectors above
names(tested_cases) <- countries
names(positive_cases) <- countries
names(active_cases) <- countries
names(hospitalized_cases) <- countries
```

```{r}
#find the ratio of positive cases
positive_cases / tested_cases
positive_tested_top_3 <- c("United Kingdom" = 0.11, "United States" = 0.10, "Turkey" = 0.08)
```

```{r}
#Create matrix with the top 3 countries with most positives from testing 
united_kingdom <- c(0.11,1473672,166909,0,0)
united_states <- c(0.10,17282363,1877179,0,0)
turkey <- c(0.08,2031192,163941,2980960,0)
covid_mat <- rbind(united_kingdom, united_states, turkey)
names_covid_mat <- c("ratio", "tested", "positive", "active", "hospitalized")
colnames(covid_mat) <- names_covid_mat
covid_mat
```

```{r}
#storing the question and answer in vectors
question <- "Which countries have had the highest number of positive cases against the number of tests"
answer <- c("positive tested cases" = positive_tested_top_3)
```


```{r}
#putting it together
dataframes <- list(original = covid_df,all_states =  covid_df_all_states, daily = covid_df_all_states_daily, top_10 = covid_top_10)
matrices <- list(covid_mat)
vectors <- list(vector_cols, countries)
data_structure_list <- list("dataframes" = dataframes, "matrices" = matrices, "vectors" = vectors)
covid_analysis_list <- list(question, answer, data_structure_list)
```


```{r}
#testing list
covid_analysis_list[[2]]
```

#Conclusion

The top 3 countries with the highest number of positive cases against the number of total test are United Kingdom, United States, and Turkey. For specifics the ratios are listed above.
