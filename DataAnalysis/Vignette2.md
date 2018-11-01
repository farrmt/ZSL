Vignette 2
================

#### Components of recruitment

To investigate which components of recruitment most strongly correlated with clan growth, we calculated clan growth as the ratio of individuals at least one year old that were present in the clan at year t to individuals over one year old at year t-1. We correlated this value with fecundity (number of cubs born in year t-1), cub survival (proportion of cubs born in year t-1 that survive to 1 year old in year t), and subadult survival (proportion of 1 year olds in year t-1 that survived to 2 years old in year t).

##### Code

``` r
yearly.demo.builder <- list()
counter <- 1

for(c.clan in clans.of.interest){
  years <- tblHyenasPerSession %>% filter(clan == c.clan) %>%
    group_by(year) %>% summarize(min = range(lubridate::month(date))[1], 
                                 max = range(lubridate::month(date))[2]) %>%
    filter(min == 1 & max == 12) %>%
    pull(year)
  
  for(c.year in years){
    ##Hyenas whose date of birth has them turning 1 in current year
    cub.dob <- filter(tblHyenas, format(birthdate + 365, '%Y') == c.year,
                      clan == c.clan)
    ##Hyenas whose date of birth has them turning 1 in current year and they 
    ##survived to 1 year old
    cub.dob1 <- filter(cub.dob, is.na(disappeared) | disappeared >= (birthdate + 365))
    
    ##Hyenas whose date of birth has them turning 2 in current year and they survived
    ##to 1 last year. 
    last.year <- as.character(as.numeric(c.year)-1)
    cub.dob1.lastyear <- filter(tblHyenas, format(birthdate + 365, '%Y') == last.year,
                                clan == c.clan, 
                                is.na(disappeared) | disappeared >= (birthdate+365))
    cub.dob2 <- filter(cub.dob1.lastyear, is.na(disappeared) | disappeared >= (birthdate + 2*365))
    
    ##Individuals who were adults last year
    adults <- filter(tblHyenas, clan == c.clan,
                     is.na(disappeared) | disappeared > paste0(last.year,'-12-31'),
                     (is.na(birthdate) & first.seen + 365*2 <= paste0(last.year,'-12-31')) | birthdate + 365*2 <= paste0(last.year,'-12-31'),
                     sex=='f', status != 't')
    ##Individuals who were adults last year and survive through the end of this year
    adults.survive <- filter(adults, is.na(disappeared) | disappeared > paste0(c.year,'-12-31'))
    
    
    ##Clan size - total individuals alive that year (over 1)
    all.clan <- filter(tblHyenas, clan == c.clan,
                       birthdate <= paste0(last.year,'-01-01') | 
                         is.na(birthdate) & first.seen < paste0(c.year,'-01-01'),
                       id %in% filter(tblHyenasPerSession, year == c.year,
                                      clan == c.clan)$hyena,
                       status != 't')
    
    ##Clan size from previous year
    all.clan.last.year <- filter(tblHyenas, clan == c.clan,
                                 birthdate <= paste0(as.numeric(last.year)-1,'-01-01') | 
                                   is.na(birthdate) & first.seen < paste0(last.year,'-01-01'),
                                 id %in% filter(tblHyenasPerSession, year == last.year,
                                                clan == c.clan)$hyena,
                                 status != 't')
 
    
    
    yearly.demo.builder[[counter]] <- data.frame(year = c.year,
                                                 clan = c.clan,
                                                 clan.size = nrow(all.clan),
                                                 clan.size.change = nrow(all.clan)/nrow(all.clan.last.year),
                                                 cub.survival = nrow(cub.dob1)/nrow(cub.dob),
                                                 sub.survival = nrow(cub.dob2)/nrow(cub.dob1.lastyear),
                                                 adult.survival = nrow(adults.survive)/nrow(adults),
                                                 fecundity = nrow(cub.dob),
                                                 cub.dob1 = nrow(cub.dob1),
                                                 cub.dob1.lastyear = nrow(cub.dob1.lastyear),
                                                 cub.dob2 = nrow(cub.dob2),
                                                 adults.t.minus.1 = nrow(adults),
                                                 adults.t = nrow(adults.survive))
    counter<-counter+1
    
  }
}
yearly.demo <- do.call(rbind, yearly.demo.builder)

##Exclude initial years for each clan when we were establishing demographic relationships
yearly.demo <- filter(yearly.demo, adults.t.minus.1 >= 5)
```

###### Talek recruitment correlations

    ##                  clan.size.change   fecundity cub.survival sub.survival
    ## clan.size.change        1.0000000  0.24276933   0.18083320    0.4286650
    ## fecundity               0.2427693  1.00000000  -0.07866122    0.1400983
    ## cub.survival            0.1808332 -0.07866122   1.00000000    0.3470574
    ## sub.survival            0.4286650  0.14009830   0.34705736    1.0000000

###### Serena recruitment correlations

    ##                  clan.size.change    fecundity cub.survival sub.survival
    ## clan.size.change       1.00000000  0.170963768 -0.081537916   0.10223117
    ## fecundity              0.17096377  1.000000000 -0.005908091  -0.03119013
    ## cub.survival          -0.08153792 -0.005908091  1.000000000  -0.18009995
    ## sub.survival           0.10223117 -0.031190126 -0.180099955   1.00000000