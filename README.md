# Code for the main analysis of the paper "Protection of BNT162b2 Vaccine Booster against Covid-19 in Israel"

### Data

The data for the main analysis of documented infection can be found in the file `PositiveDataAug10-31.csv` and for severe illness in the file `SevereDataAug10-26.csv`. 
To read the data into the R Statistical Software, first load the libraries `tidyverse`, `pander` and `broom` and then use the following code. Note that cells that have less than 5 individuals are omitted from the analysis.
```
dat_positive <- read_csv("PositiveDataAug10-31.csv") %>% 
  filter(N_person	!= "<5") %>%
  mutate(`Vacc Period` = factor(`Vacc Period`,levels = c("JanB","FebA","FebB")), 
         Cohort = factor(Cohort,levels= c("2nd","12+")),
         `Epi Day` = factor(`Epi Day`),
         Sector = factor(Sector, levels = c("General Jewish", "Arab","ultra-Orthodox Jews")),
         N_person = as.numeric(N_person),
        Positive =  Rate_per_1K*N_person/1000)

dat_severe <- read_csv("SevereDataAug10-26.csv") %>%  
  filter(N_person	!= "<5") %>%
  mutate(`Vacc Period` = factor(`Vacc Period`,levels = c("JanB","FebA","FebB")), 
         Cohort = factor(Cohort,levels= c("2nd","12+")),
         `Epi Day` = factor(`Epi Day`),
         Sector = factor(Sector, levels = c("General Jewish", "Arab","ultra-Orthodox Jews")),
          N_person = as.numeric(N_person),
        Severe =  Rate_per_1K*N_person/1000)
```

### Analysis

To run the main analysis of the paper for both confirmed infections and severe illness use the following code. 

```
formula_positive <- as.formula("Positive ~  Age +
                         `Vacc Period` +
                         Gender + 
                         `Epi Day` +
                         Sector +
                         Cohort +  
                         offset(log(N_person))") 


formula_severe <- as.formula("Severe ~  Age +
                         `Vacc Period` +
                         Gender + 
                         `Epi Day` +
                         Sector +
                         Cohort +  
                         offset(log(N_person))") 

analysis_positive <- glm(formula_positive, family="poisson", data=dat_positive)
analysis_severe <- glm(formula_severe, family="poisson", data=dat_severe)

```

### Confidence Intervals

To calculate a 95% confidence interval for confirmed infections use the following code:
```


df <- broom::tidy(analysis_positive) %>% 
  filter(term == "Cohort12+") %>% 
  mutate(est =  round(exp(-estimate),1),
         lower = round(exp(-estimate-1.96*std.error),1),
         upper = round(exp(-estimate+1.96*std.error),1),
         `Confidence Interval: Positive`= paste0(est," 95% CI: [",lower,", ",upper,"]")) %>% 
  select(`Confidence Interval: Positive`)

pander::pander(df)
```


To calculate a 95% confidence interval for severe illness use the following code:
```


df <- broom::tidy(analysis_severe) %>% 
  filter(term == "Cohort12+") %>% 
  mutate(est =  round(exp(-estimate),1),
         lower = round(exp(-estimate-1.96*std.error),1),
         upper = round(exp(-estimate+1.96*std.error),1),
         `Confidence Interval: Severe`= paste0(est," 95% CI: [",lower,", ",upper,"]")) %>% 
  select(`Confidence Interval: Severe`)

pander::pander(df)
```
