---
title: "Homework 3"
author: "Jill Guerra"
date: "September 29, 2015"
output: 
  html_document:
    keep_md: true 
---

Exploring Inequality in the Americas 
---
While confined to a few demographic variables - population, GDP per capita and life expectancy - this homework is a basic exploration of the changes in inequality in the Americas over time as well as between and within regions in the Americas. 

_Setting up the analysis_<br>
Note: see raw file for full code 
```{r, results='hide', message=FALSE, warning=FALSE, include=FALSE}
#install.packages("dplyr")
library(dplyr)
library(gapminder)
library(ggplot2)
gtbl <- tbl_df(gapminder) # assign data.frame to a tbl_df
gtbl
```
* Files were loaded
* Data broken up into regional groups looking at North, Central and South America (FYI Caribbean countries not included)
```{r regions, include=FALSE}
America <- gtbl %>% filter(continent == "Americas")
NorthAmer <- c("Canada", "United States", "Mexico")
CentAmer <- c("Guatemala","El Salvador", "Honduras", "Nicaragua", "Costa Rica", "Panama")
SouthAmer <- c("Paraguay", "Uruguay", "Colombia", "Venezuela", "Ecuador", "Peru", "Chile", "Argentina", "Brazil", "Bolivia")
```

 
```{r regiondf, include=FALSE}
# North America 
nale <- gtbl %>% 
  filter(country %in% NorthAmer) %>% 
  arrange(year) %>% 
  group_by(year)

# Central America 
cale <- gtbl %>% 
  filter(country %in% CentAmer) %>% 
  arrange(year) %>% 
  group_by(year)

# South America 
sale <- gtbl %>% 
  filter(country %in% SouthAmer) %>% 
  arrange(year) %>% 
  group_by(year)
```
<br>

Inequality in Life Expectancy in the Americas 
--- 

To investigate the inequality in life expectancy, the  following section uses a crude measure of inequality in life expectancy: difference between the min and max life expectancy in each region per year. The data is summarized in the following table and then presented in a graph.  
```{r regiondiff, include=FALSE}
# creates new data frames 
nadiff <- nale %>% 
  arrange(year) %>% 
  group_by(year) %>% 
  filter(min_rank(desc(lifeExp)) < 2 | min_rank(lifeExp) < 2) %>% 
  arrange(desc(lifeExp), year)

cadiff <- cale %>% 
  arrange(year) %>% 
  group_by(year) %>% 
  filter(min_rank(desc(lifeExp)) < 2 | min_rank(lifeExp) < 2) %>% 
  arrange(desc(lifeExp), year)

sadiff <- sale %>% 
  arrange(year) %>% 
  group_by(year) %>% 
  filter(min_rank(desc(lifeExp)) < 2 | min_rank(lifeExp) < 2) %>% 
  arrange(desc(lifeExp), year)

# find the inequality in life expectancy for each region by year. Create new df to store these then merge all together to create ggplot  
diff_le <- nadiff %>% 
  mutate(diff_le = lifeExp - lead(lifeExp)) %>% 
  mutate(region = "North America") %>% 
  select(country, year, diff_le, region)
    
diff_le2 <- cadiff %>% 
  mutate(diff_le = lifeExp - lead(lifeExp)) %>% 
  mutate(region = "Central America") %>% 
  select(country, year, diff_le, region)

diff_le3 <- sadiff %>% 
  mutate(diff_le = lifeExp - lead(lifeExp)) %>% 
  mutate(region = "South America") %>% 
  select(country, year, diff_le, region)


newdiff_le<- rbind(diff_le, diff_le2)
fulldiff_le <- rbind(newdiff_le, diff_le3)
odiff_le <- na.omit(fulldiff_le) # omits the rows where 'na' was present. 
odiff_le$country <- NULL
```

```{r plot differences}

t <- ggplot(odiff_le, aes(x = year, y = diff_le, color = region)) + geom_line() + geom_point() 

t + xlab("Year") + ylab("Inequality in life Expectancy") 
```
<div class="twoC">
```{r results = 'asis', echo=FALSE}
odiff_sum <- odiff_le %>%
  group_by(region) %>%
  summarise_each(funs(mean, median), diff_le)
knitr::kable(odiff_sum)
```

Thoughts:
---

__Life Expectancy__:
The above graphs clearly indicate that life expectancy has been increasing across the Americas over time. What is also immediately apparent is the tendancy for life expectancy to converge over time. For example, in North America the life expectancy of Mexicans began much lower at around 50 years in 1950 but increased dramatically to around 75 years - a gain of 25 years. However, Canada only gained about 14 years (from around 68 to 82 years). The same trend is visible in Central American and South America. 
<br><br>
__GDP per Capita__:
GDP per Capita seems to increase with life expectancy, particularly for countries near the higher range of life expectancy (i.e. Canada, United States, Costa Rica). Interestingly, life expectancy for countries with low GDP per capita still have been able to improve their life expectancy outcomes. <br>
To explore this further, the following graphs show the within region spread of life expectancy over time.  

Regional Graphs of Inequality in Life Expectancy in the Americas
---
```{r}

n <- ggplot(nale, aes(x = year, y = lifeExp)) 
n + geom_point(aes(size = gdpPercap, color = country)) + geom_smooth(lwd = 0.5, color="black", method=loess, se = FALSE) + ylim(40, 85) +ggtitle("Life Expectancy in North America")

c <- ggplot(cale, aes(x = year, y = lifeExp)) 
c + geom_point(aes(size = gdpPercap, color = country)) + geom_smooth(lwd = 0.5, color="black", method=loess, se = FALSE) + ylim(40, 85) +ggtitle("Life Expectancy in Central America")

s <- ggplot(sale, aes(x = year, y = lifeExp)) 
s + geom_point(aes(size = gdpPercap, color = country)) + geom_smooth(lwd = 0.5, color="black", method=loess, se = FALSE) + ylim(40, 85) +ggtitle("Life Expectancy in South America")
```
Thoughts: 
---
__Bolivia__:
Bolivia is a main driver of inequality in life expectancy in South America according to the measure being used. The country has a low level of GDP per capita and has remained much behind in increases in life expectancy. 
<br><br>
__El Salvador & Guatemala__:
Central America's spike in inequality in life expectancy in the 1979 and 1982 seem to be largely driven by El Salvador & Guatemala, as shown in the regional graph for Central America.  This can be explained fairly easily: El Salvador was in the grips of a [brutal civil war](https://en.wikipedia.org/wiki/Salvadoran_Civil_War) from 1979-1992. While less dramatic in the data, Guatemala's civil war had a similar effect.


Brazil as a Benchmark 
---

As one of the 'BRICS', Brazil is considered an emerging economy - something like a teenager waiting to graduate to full 'development' status. For this next exploration, Brazil will be used as a benchmark to measure the changes in life expectancy across the Americas. 

```{r, include=FALSE}
just_brazil <- America %>% 
  filter(country == "Brazil")
sa <- America %>% 
  filter(country %in% SouthAmer) %>% 
  mutate(region = "South America")
na <- America %>% 
  filter(country %in% NorthAmer) %>% 
  mutate(region = "North America")
ca <- America %>% 
  filter(country %in% CentAmer) %>% 
  mutate(region = "Central America")

America1 <- rbind(sa, na)
Americafull <- rbind(America1, ca)

America_final <- Americafull %>%
  mutate(brazil = rep(just_brazil$lifeExp, 19),
  rel_le = lifeExp / brazil) # 25 was chosen because 19 obs in just_brazil, but need to repeat that 25 times to reach the 228 obs in the America df.
```

```{r}
brazil_base <- ggplot(America_final, aes(x = year, y = rel_le, color = region))
brazil_base + geom_point() + geom_hline(yintercept = 1)

brazil_base2 <- ggplot(America_final, aes(x = year, y = rel_le, color = country))
brazil_base2 + geom_point() + geom_hline(yintercept = 1) + geom_line()
```
<div class="twoC">
```{r results = 'asis', echo=FALSE}
America_final_sum <- America_final %>%
  group_by(region, country) %>%
  summarise_each(funs(mean, median), rel_le)
knitr::kable(America_final_sum)
```

Spread of GDP per capita in the Americas 
---


``` {r}
gdp_spread<-America_final %>%
  group_by(region,country) %>%
  summarise(max_GDPpercap=round(max(gdpPercap),0),min_GDPpercap=round(min(gdpPercap),0),    mean_GDPpercap=round(mean(gdpPercap),0))

knitr::kable(gdp_spread)

g <- ggplot(gdp_spread, aes(x=country,y=mean_GDPpercap,color=region))
g+geom_point()+geom_errorbar(aes(ymax=max_GDPpercap,ymin=min_GDPpercap), width=0.3)+geom_text(aes(label=gdp_spread$country,fontface=2),size=2)+ylab("Spread of GDP per Capita")+theme(axis.text.x=element_blank(), axis.ticks=element_blank())

```

Reflections: 

- I struggled to find a simple way to calculate the difference between the min and max of a variable, per year, by the regions I created. I did it the 'brute force' which took a lot of time and space. I am assuming there are easier ways to do this,  but I am currently unaware of them! 
- I feel like I used too many different data frames, next time I think I will explore just filtering larger dataframes to use rather than creating new ones based on the select() funciton. 
- It has been neat exploring the different specifications of the ggplots. In previous assignments I did make as many modifications as I did this time. 
