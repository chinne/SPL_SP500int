[<img src="https://github.com/QuantLet/Styleguide-and-Validation-procedure/blob/master/pictures/banner.png" alt="Visit QuantNet">](http://quantlet.de/index.php?p=info)

## [<img src="https://github.com/QuantLet/Styleguide-and-Validation-procedure/blob/master/pictures/qloqo.png" alt="Visit QuantNet">](http://quantlet.de/) **SPL_SP500int_Panel Data Analysis** [<img src="https://github.com/QuantLet/Styleguide-and-Validation-procedure/blob/master/pictures/QN2.png" width="60" alt="Visit QuantNet 2.0">](http://quantlet.de/d3/ia)

```yaml

Name of QuantLet: SPL_SP500int_Panel Data Analysis

Published in: has not been published

Description: 'Description: In order to see in which direction is the return of stocks 
in internet and software industry moves with respect to the whole market, we run pooled 
regression of these stock returns on the market return,  and test whether the error 
terms has heterogeneous variance and serial correlations. Also we use robust variance 
and run the regression again.'

See also: SPL_SP500int_Data Description, SPL_SP500int_CAPM Regression

Author: Wei Zhang, Yue Wang, Jinhua Yang, Fan Song

Datafile: Panel.csv

Output: data summary, plots

Example: scatterplot of the data and regression results.

```

![Picture1](pooled regression results.png)
![Picture1](pooled regression.png)
![Picture1](Density of residuals.png)
![Picture1](Q-Q plot.png)
![Picture1](Comparison of Robust and OLS.png)
![Picture1](Fixed Effects Model Regression.png)
![Picture1](Random Effects Model Regression.png)
![Picture1](comparison of OLS and LSDV.png)
![Picture1](Robust OLS.png)



```r
# clear history
rm(list = ls(all = TRUE))
graphics.off()

# set working directory setwd('...')

# Panel Data Analysis

# Pooled Regression Model

# regression

# install.packages('Formula') install.packages('plm') install.packages('stargazer')
library(Formula)
library(plm)
library(stargazer)
Paneldata = read.csv("Panel.csv")
Ri = Paneldata$Ri
Rm = Paneldata$Rm
pooling = plm(Ri ~ Rm, data = Paneldata, index = c("company", "date"), model = "pooling")
summary(pooling)
stargazer(pooling, title = "Pooled Regression Results")

plot(Ri ~ Rm, data = Paneldata, main = "relationship between market return and stock return", 
    xlab = "market return", ylab = "stock return")
model = lm(Ri ~ Rm)
abline(model, col = "red")

# Residual Analysis Empirical Density of Residuals

res = pooling$resid
dens_res = density(res)
plot(dens_res)
plot(res, ylab = "residuals(pooling)")
qqnorm(res)
qqline(res)
# normality assumption is satiesfied

# Residuals and Explanatory Variable
par(mfrow = c(1, 1))
plot(Paneldata$Rm, res)
model1 = lm(res ~ Paneldata$Rm)
abline(model1, col = "red")

y_pred = predict(pooling)
plot(y_pred, res)
model2 = lm(res ~ y_pred)
abline(model2, col = "blue")

# heterogeneity testing install.packages('lmtest') install.packages('zoo')
library(zoo)
library(lmtest)
bptest(pooling)

# serial correlation testing
pbgtest(pooling)

# Use Robust Regression install.packages(('MASS'))
library(MASS)
rols = rlm(Ri ~ Rm)
summary(rols)
stargazer(rols, title = "Robust OLS Results", align = TRUE)

# install.packages('apsrtable')
library(apsrtable)
OLS = lm(Ri ~ Rm)
coefficients(OLS)
coefficients(rols)
stargazer(OLS, rols, title = "regression of standard OLS and robust OLS")

#install.packages("car")
# Fixed Effect Regression
#install.packages("lattice")

library(lattice)
library(car)

com = factor(Paneldata$company)
xyplot(Ri ~ Rm | com, data = Paneldata, layout = c(10, 1), xlab = "market return", 
    ylab = "stock return", main = "stock return and market return")
xyplot(Ri ~ Rm | com, data = Paneldata, panel = function(x, y) {
    panel.xyplot(x, y)
    panel.abline(h = median(y), lty = 2, col = "gray")
    panel.lmline(x, y, col = "red")
}, xlab = "market return", ylab = "stock return", main = "relationship between market return and stock return")

fixed.dum = lm(Ri ~ Rm + factor(com) - 1, data = Paneldata)
summary(fixed.dum)
stargazer((fixed.dum), title = "Fixed Effects Regression Using Dummy Variables")

# Significance of Fixed Effects
yhat = fixed.dum$fitted.values
library(car)
scatterplot(yhat ~ Rm | Paneldata$company, boxplots = FALSE, xlab = "Market price", 
    ylab = "yhat", smooth = FALSE)
abline(lm(Ri ~ Rm), lwd = 3, col = "Dark Blue")

# Using plm
library(plm)
fixed = plm(Ri ~ Rm, data = Paneldata, index = c("company", "date"), model = "within")
summary(fixed)
stargazer(fixed, title = "Fixed Effects Regression Using plm")

# Comparison of Fixed Effects and OLS
library(apsrtable)
apsrtable(OLS, fixed.dum, model.names = c("OLS", "Fixed Effects"))

# Testing for Fixed Effects
test_fixed = pFtest(fixed, pool)
stargazer(test_fixed)

# Random Effect Regression
random = plm(Ri ~ Rm, data = Paneldata, index = c("company", "date"), model = "random")
summary(random)
stargazer((random), title = "Random Effects Regression")

# Test for the Difference between Fixed Effect Model and Random Effect Model
phtest(fixed, random)

# Comparison of Random Effects Model and Pooled Regression Model
plmtest(pooling)
# In summary, use pooled regression model.

```