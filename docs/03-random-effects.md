# Random effects {#random-effects}

![](images/randomeffectlines.png)



### In brief

> In this session we specify more complex random effects, including examples from
> experimental data with multiple sources of variation. We use visualisation techniques
> to explore the concept of 'shrinkage'.



## Politeness 

Winter & Grawunder, 2012 describe a study of the pitch (frequency) of individuals' speech when they recorded different phrases (scenarios). The scenarios differed in whether they required 'politeness' or were more informal in nature. A reduced version of this dataset is also described and analysed in a mixed-models tutorial (see http://www.bodowinter.com/tutorial/bw_LME_tutorial2.pdf).

The data are available online and can be read from this URL:



```r
# this previously taken from http://www.bodowinter.com/tutorial/politeness_data.csv but that link now a 404
politeness <-  read_csv("https://raw.githubusercontent.com/opetchey/BIO144/master/3_datasets/politeness_data.csv")
```


:::{.exercise}

1. Make a plot showing how average levels of frequency differs between individuals.




<div class='solution'><button class='solution-button'>Show me the plot</button>



```r
politeness %>%
  ggplot(aes(subject, frequency)) + geom_boxplot()
```

<img src="03-random-effects_files/figure-html/unnamed-chunk-3-1.png" width="672" />

You could also use `stat_summary()` instead of `geom_boxplot()` to get a point range plot.


</div>



2. Make a plot showing how frequency differs between scenarios



<div class='solution'><button class='solution-button'>Show me the plot</button>



```r
politeness %>%
  ggplot(aes(factor(scenario), frequency)) + geom_boxplot()
```

<img src="03-random-effects_files/figure-html/unnamed-chunk-4-1.png" width="672" />


</div>



3. Based on the plots, which do you think accounts for more variation in frequency: `subject` or `scenario`?



<div class='solution'><button class='solution-button'>Show answer</button>


It looks like subjects vary more in frequency than do scenarios, but there appears to be variation attributable to both variables.


</div>




4. Fit a random intercepts model to the data, allowing for variation between subjects.


<div class='solution'><button class='solution-button'>Show me that model</button>



```r
(polite.ri.subject <- lmer(frequency ~ attitude + (1|subject), data=politeness))
```

```
## Linear mixed model fit by REML ['lmerModLmerTest']
## Formula: frequency ~ attitude + (1 | subject)
##    Data: politeness
## REML criterion at convergence: 804.7023
## Random effects:
##  Groups   Name        Std.Dev.
##  subject  (Intercept) 63.10   
##  Residual             29.17   
## Number of obs: 83, groups:  subject, 6
## Fixed Effects:
## (Intercept)  attitudepol  
##      202.59       -19.38
```



</div>




<div class='solution'><button class='solution-button'>What other commands should I run to interpret the model?</button>

You might then also want to look at the regression coefficients, Anova table, and tests of random effects:


```r
# regression coefficients
polite.ri.subject %>% summary %>% coef()
```

```
##              Estimate Std. Error        df   t value     Pr(>|t|)
## (Intercept) 202.58810  26.151105  5.150024  7.746827 0.0005008339
## attitudepol -19.37584   6.406962 76.002550 -3.024185 0.0033990395
```

```r
# anova-table
anova(polite.ri.subject)
```

```
## Type III Analysis of Variance Table with Satterthwaite's method
##          Sum Sq Mean Sq NumDF  DenDF F value   Pr(>F)   
## attitude 7782.9  7782.9     1 76.003  9.1457 0.003399 **
## ---
## Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1
```

```r
# tests of random effects
ranova(polite.ri.subject)
```

```
## ANOVA-like table for random-effects: Single term deletions
## 
## Model:
## frequency ~ attitude + (1 | subject)
##               npar  logLik   AIC   LRT Df Pr(>Chisq)    
## <none>           4 -402.35 812.7                        
## (1 | subject)    3 -457.15 920.3 109.6  1  < 2.2e-16 ***
## ---
## Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1
```



</div>




4. Fit a second random intercepts model, allowing for variation in both subjects and between the different scenarios. Use to `VarCorr` function to estimate how much variation (what percentage of the total) was between subjects, and what percentage was between scenarios.


<div class='solution'><button class='solution-button'>Show me that model</button>



```r
polite.ri.both <- lmer(frequency ~ attitude + 
                         (1|subject) + (1|scenario), data=politeness)
ranova(polite.ri.both)
```

```
## ANOVA-like table for random-effects: Single term deletions
## 
## Model:
## frequency ~ attitude + (1 | subject) + (1 | scenario)
##                npar  logLik    AIC     LRT Df Pr(>Chisq)    
## <none>            5 -396.73 803.45                          
## (1 | subject)     4 -457.15 922.30 120.851  1  < 2.2e-16 ***
## (1 | scenario)    4 -402.35 812.70  11.249  1  0.0007968 ***
## ---
## Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1
```


</div>




5. How much variance was attributable to subjects or scenarios, compared with the total variance?


<div class='solution'><button class='solution-button'>Show ne how to calculate this</button>


Remember to convert the `VarCorr` output to a dataframe to see the variance (rather than standard deviations) in the `vcov` column. You can then use `mutate` to calculate the within/betwen ratio (the ICC).


```r
VarCorr(polite.ri.both) %>%  
  as.tibble() %>% 
  select(grp, vcov) %>% 
  mutate(icc = vcov /  sum(vcov)) %>% 
  pander()
```


---------------------------
   grp      vcov     icc   
---------- ------ ---------
 scenario   219    0.04488 

 subject    4015   0.8227  

 Residual   646    0.1324  
---------------------------





</div>



6. Add random slopes to the model above. Allow the effect of `attitude` (whether the scenario was polite or informal) to differ between subjects, and also to differ between the different example scenarios sampled in this study.



<div class='solution'><button class='solution-button'>Show me that model formula</button>



```r
frequency ~ attitude + (attitude|subject) + (attitude|scenario)
```

```
## frequency ~ attitude + (attitude | subject) + (attitude | scenario)
```


</div>



7. Use `ranova` to test whether the random slopes were significant predictors of variance in `frequency`.



<div class='solution'><button class='solution-button'>Show me how</button>


First run the model:


```r
polite.rslopes <- lmer(frequency ~ attitude + (attitude|subject) +
                         (attitude|scenario), data=politeness)
```



:::{.tip}

If you see a message saying `boundary (singular) fit: see ?isSingular` don't worry for now. This IS actually quite important, but we will discuss it in more detail in the next session.

:::


And then use `ranova`:


```r
ranova(polite.rslopes)
```

```
## ANOVA-like table for random-effects: Single term deletions
## 
## Model:
## frequency ~ attitude + (attitude | subject) + (attitude | scenario)
##                                   npar  logLik    AIC     LRT Df Pr(>Chisq)
## <none>                               9 -395.76 809.51                      
## attitude in (attitude | subject)     7 -396.55 807.11 1.59356  2     0.4508
## attitude in (attitude | scenario)    7 -395.95 805.89 0.38097  2     0.8266
```

Both of the `Pr(>Chisq)` values (the *p* values for the chi squared test) are non-significant.


</div>




7. Use `anova` to test the same thing: whether adding the random slope for `attitude` within `scenario` or `subject` improved the model.



<div class='solution'><button class='solution-button'>Show me how</button>


First run two models, with and without the random slope you want to test:


```r
model.without <- lmer(frequency ~ attitude + 
                         (1|subject) +
                         (attitude|scenario), data=politeness)


model.with <- lmer(frequency ~ attitude + 
                         (attitude|subject) +
                         (attitude|scenario), data=politeness)
```

And then use `anova`:


```r
anova(model.with, model.without)
```

```
## Data: politeness
## Models:
## model.without: frequency ~ attitude + (1 | subject) + (attitude | scenario)
## model.with: frequency ~ attitude + (attitude | subject) + (attitude | scenario)
##               Df    AIC    BIC  logLik deviance  Chisq Chi Df Pr(>Chisq)
## model.without  7 820.88 837.81 -403.44   806.88                         
## model.with     9 823.32 845.09 -402.66   805.32 1.5578      2     0.4589
```

You should notice that the  *p* value for the test has the same number of degrees of freedom, and the same value (very slightly different due to rounding errors).

The `Pr(>Chisq)` is again non-significant, indicating that the random slope within `subject` doesn't explain additional variance.


</div>



8. If a random slopes model is not 'significantly' better than a similar model which does not include the random slope term, is there any reason why we  might still prefer it, and use the 'full' model to base our inference on?



<div class='solution'><button class='solution-button'>Show answer</button>


Yes - simulation studies, including Barr et al 2013 suggest that a 'maximal' model is likely to be more conservative than a model which excludes non-significant random effects terms.


</div>




9. How might mixed models make the problem of 'researcher degrees of freedom' worse? What effect might this have? How can it be avoided?


<div class='solution'><button class='solution-button'>Show answer</button>


Mixed models provide many more possible 'ways to do it'. In addition to different fixed-effects specifications we can now also have many different random effects specifications. This increases degrees of freedom in the analysis, and makes it even more important to pre-specify analyses.


</div>



:::





<!---


```r
polite.ri1 <- lmer(frequency ~ attitude + (1|subject), data=politeness)
summary(polite.ri1)
```

```
## Linear mixed model fit by REML. t-tests use Satterthwaite's method [
## lmerModLmerTest]
## Formula: frequency ~ attitude + (1 | subject)
##    Data: politeness
## 
## REML criterion at convergence: 804.7
## 
## Scaled residuals: 
##     Min      1Q  Median      3Q     Max 
## -2.2953 -0.6018 -0.2005  0.4774  3.1772 
## 
## Random effects:
##  Groups   Name        Variance Std.Dev.
##  subject  (Intercept) 3982     63.10   
##  Residual              851     29.17   
## Number of obs: 83, groups:  subject, 6
## 
## Fixed effects:
##             Estimate Std. Error      df t value Pr(>|t|)    
## (Intercept)  202.588     26.151   5.150   7.747 0.000501 ***
## attitudepol  -19.376      6.407  76.003  -3.024 0.003399 ** 
## ---
## Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1
## 
## Correlation of Fixed Effects:
##             (Intr)
## attitudepol -0.121
```

```r
ranova(polite.ri1)
```

```
## ANOVA-like table for random-effects: Single term deletions
## 
## Model:
## frequency ~ attitude + (1 | subject)
##               npar  logLik   AIC   LRT Df Pr(>Chisq)    
## <none>           4 -402.35 812.7                        
## (1 | subject)    3 -457.15 920.3 109.6  1  < 2.2e-16 ***
## ---
## Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1
```


```r
polite.ri <- lmer(frequency ~ attitude + (1|subject) + (1|scenario), data=politeness)
summary(polite.ri)
```

```
## Linear mixed model fit by REML. t-tests use Satterthwaite's method [
## lmerModLmerTest]
## Formula: frequency ~ attitude + (1 | subject) + (1 | scenario)
##    Data: politeness
## 
## REML criterion at convergence: 793.5
## 
## Scaled residuals: 
##     Min      1Q  Median      3Q     Max 
## -2.2006 -0.5817 -0.0639  0.5625  3.4385 
## 
## Random effects:
##  Groups   Name        Variance Std.Dev.
##  scenario (Intercept)  219     14.80   
##  subject  (Intercept) 4015     63.36   
##  Residual              646     25.42   
## Number of obs: 83, groups:  scenario, 7; subject, 6
## 
## Fixed effects:
##             Estimate Std. Error      df t value Pr(>|t|)    
## (Intercept)  202.588     26.754   5.575   7.572 0.000389 ***
## attitudepol  -19.695      5.585  70.022  -3.527 0.000748 ***
## ---
## Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1
## 
## Correlation of Fixed Effects:
##             (Intr)
## attitudepol -0.103
```

```r
VarCorr(polite.ri)
```

```
##  Groups   Name        Std.Dev.
##  scenario (Intercept) 14.798  
##  subject  (Intercept) 63.361  
##  Residual             25.417
```

```r
ranova(polite.ri)
```

```
## ANOVA-like table for random-effects: Single term deletions
## 
## Model:
## frequency ~ attitude + (1 | subject) + (1 | scenario)
##                npar  logLik    AIC     LRT Df Pr(>Chisq)    
## <none>            5 -396.73 803.45                          
## (1 | subject)     4 -457.15 922.30 120.851  1  < 2.2e-16 ***
## (1 | scenario)    4 -402.35 812.70  11.249  1  0.0007968 ***
## ---
## Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1
```




```r
politeness.agg <- politeness %>% 
  group_by(subject, attitude) %>% 
  summarise(frequency = mean(frequency))
  
polite.ri.agg <- lmer(frequency ~ attitude + (1|subject), data=politeness)
polite.ri %>% summary %>%  coefficients
```

```
##              Estimate Std. Error        df   t value     Pr(>|t|)
## (Intercept) 202.58810  26.753654  5.575042  7.572352 0.0003892701
## attitudepol -19.69454   5.584683 70.022146 -3.526528 0.0007475343
```

```r
polite.ri.agg %>% summary %>%  coefficients
```

```
##              Estimate Std. Error        df   t value     Pr(>|t|)
## (Intercept) 202.58810  26.151105  5.150024  7.746827 0.0005008339
## attitudepol -19.37584   6.406962 76.002550 -3.024185 0.0033990395
```




```r
polite.rs <- lmer(frequency ~ attitude + 
                    (attitude|subject) + 
                    (attitude|scenario), data=politeness)
polite.rs %>% summary %>%  coefficients
```

```
##              Estimate Std. Error       df   t value     Pr(>|t|)
## (Intercept) 202.58810  28.204932 5.376367  7.182719 0.0005975745
## attitudepol -19.66341   7.058335 6.280892 -2.785842 0.0302626661
```

```r
polite.rs %>% ranova
```

```
## ANOVA-like table for random-effects: Single term deletions
## 
## Model:
## frequency ~ attitude + (attitude | subject) + (attitude | scenario)
##                                   npar  logLik    AIC     LRT Df Pr(>Chisq)
## <none>                               9 -395.76 809.51                      
## attitude in (attitude | subject)     7 -396.55 807.11 1.59356  2     0.4508
## attitude in (attitude | scenario)    7 -395.95 805.89 0.38097  2     0.8266
```



```r
anova(polite.ri, polite.rs)
```

```
## Data: politeness
## Models:
## polite.ri: frequency ~ attitude + (1 | subject) + (1 | scenario)
## polite.rs: frequency ~ attitude + (attitude | subject) + (attitude | scenario)
##           Df    AIC    BIC  logLik deviance  Chisq Chi Df Pr(>Chisq)
## polite.ri  5 817.04 829.13 -403.52   807.04                         
## polite.rs  9 823.32 845.09 -402.66   805.32 1.7166      4     0.7877
```


-->



## Further reading



https://besjournals.onlinelibrary.wiley.com/doi/full/10.1111/j.2041-210x.2012.00261.x


Barr et al go into more detail on why mixed models are preferable to RM anova, and this paper will be a useful reference for future sessions too:  [Barr, D. J., Levy, R., Scheepers, C., & Tily, H. J. (2013). Random effects structure for confirmatory hypothesis testing: Keep it maximal. Journal of memory and language, 68(3), 255-278.](http://eprints.gla.ac.uk/79067/)

This paper expands on the original 'keep it maximal' paper, covering situations with interactions between within factors: [Barr, D. J. (2013). Random effects structure for testing interactions in linear mixed-effects models. Frontiers in psychology, 4, 328.](http://eprints.gla.ac.uk/88175/)

See also: [Eager, C., & Roy, J. (2017). Mixed effects models are sometimes terrible. arXiv preprint arXiv:1701.04858.](https://arxiv.org/abs/1701.04858)


