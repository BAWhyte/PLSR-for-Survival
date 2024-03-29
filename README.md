# PLSR-for-Survival


## I: Solution to a multicollinearity problem in ecological research

Chemical compounds on the exoskeleton of insects create a waxy waterproof layer that helps prevent water loss from their bodies. These "cuticular hydrocarbon compounds" (CHCs) are probably produced by the same biosynthetic pathways, so they are not independent variables. But if we want to know which CHCs or compound classes explain survival against water loss, we needed to treat them as separate variables in a regression model, which lead to impossibly large coefficients that cannot be trusted/interpreted. Sometimes, when covariates are so correlated with each other, regression models can't even output coefficients, and just give errors or "NA" instead. So, we need a strategy for dealing with multicollinearity. 

I read some relevant chapters from ["Regression Model Strategies"](https://link.springer.com/book/10.1007/978-3-319-19425-7), and came across the solution many have heard of before: putting the data in a PCA model, and using the principal components as the model covariates instead. Previously, I had avoided this because the interpretation of a PCA and its components is limited. A PCA allows you to examine the dimensionality of the X values, and if you show that colonies are different in certain X values, then you could suggest that maybe this difference also explained the survival differences (Y values). However, there is a version of principal component regression that not only transforms X data into different dimensions, but also estimates how those dimensions relate to the response variable (Y values).

This method is called **"Partial Least Squares Regression" (PLSR)**. It is called this because it partially involves using least squares in its formula to determine a line of best fit. An odd name, since I don't think it explains what makes it special. Here is a good description from [this website](http://www.sthda.com/english/articles/37-model-selection-essentials-in-r/152-principal-component-and-partial-least-squares-regression-essentials/): 

> *"A possible drawback of PCR [principal component regression] is that we have no guarantee that the selected principal components are associated with the outcome. Here, the selection of the principal components to incorporate in the model is not supervised by the outcome variable. An alternative to PCR is the Partial Least Squares (PLS) regression, which identifies new principal components that not only summarizes the original predictors, but also that are related to the outcome. These components are then used to fit the regression model. So, compared to PCR, PLS uses a dimension reduction strategy that is supervised by the outcome."*

Therefore, a PLS regression (PLSR) is a better tool if we want to claim that certain components or covariates drove survival in our experiments. If you want to fully understand this method and its use in ecological research like our own, I highly suggest reading [this paper](https://onlinelibrary-wiley-com.libproxy.berkeley.edu/doi/10.1111/j.1600-0706.2008.16881.x) (cited below), as it explains why PLS is useful and how to interpret its results.

*Carrascal, L. M., Galván, I., & Gordo, O. (2009). Partial least squares regression as an alternative to current regression methods used in ecology. Oikos, 118(5), 681-690.*

## II: Data frame and PLSR summary 

```{r eval=TRUE}
## Here is the structure or scale of our data, modified for use in PLSR:
dfp <- subset(data, select = c(LT50d,Avg,pL,pMo,pDi,pTri,wChain))
head(dfp)
```
LT50d is the response variable, measuring survival by lethal time at which 50% of the subjects have died. Avg refers to everage body mass, which we know influences water loss from any body. The remaining variables are CHC metrics. A list of abbreviations:

- pl = % of n-alkanes on profile
- pMo = % of mono-methyl alkanes on profile
- pDi = % of di-methyl alkanes on profile
- pTri = % of tri-methyl alkanes on profile
- wChain = Weighted average chain length of profile

Inputting this data into a PLS looks like this:

```{r eval=TRUE}
## Partial least squares regression (formula + summary)
## https://cran.r-project.org/web/packages/pls/vignettes/pls-manual.pdf
plsr_model <- plsr(LT50d ~ ., data = dfp, scale = TRUE, validation = "CV")
## LT50 vs. all covariates in dfp. Scale = TRUE scales the data, similar to the z-scoring we did before. Validation = CV is for cross-validation. Other validation options seem more specialized. 
summary(plsr_model)
```
Just like a principal components analysis, the covariates are replaced with components that are transformations of the covariates you provided. Ideally, the majority of the variance in your data can be explained with a few of these components. So the first step to interpretation is to decide how many components are needed to explain our survival data. Remember, PLSR allows us to not only see how much of the X variance is explained (body size, CHC proportions, etc.) but also how much of the Y variance is explained (LT50 from all drierite tubes) by these components. The variance explained can be seen in the summary above, but I will summarize it in the table below:

![Table1_PLSRSummary](https://user-images.githubusercontent.com/15988774/209012213-26dffe8f-e920-43ca-8e75-14593d889e00.jpg)

PC1 tells us the most about LT50, and the second component also adds some detail, but beyond that the help becomes negligible. PC2 is useful if we want to understand the dimensionality of the X variables (which is what usual PCA biplots show). RMSEP stands for "Root Mean Square Error of Prediction", and the decision of how many components to select should be judged by how minimized we can make this error prediction. Because 1.81 is as low as we can go, and we can achieve this with just the first two components, it is safe to just use these two components for our analysis and interpretation.

Each of these components is made from transformations of the covariates, and some of the covariates had more "weight" in these transformations. This part is hard to explain (some background provided in [this video](https://youtu.be/Vf7doatc2rA)), but to put it simply, if PC1 explains most of the Y variance, we can look at the "loading weights" from each covariate for PC1 in order to see which ones correlate most with PC1. Perhaps this will be easier to think of if we look at a regular PCA biplot using our data:

![XBiplot](https://user-images.githubusercontent.com/15988774/209012296-47f0e6b5-6188-459d-a39a-e5ef078e093c.jpeg)

The black numbers are the X scores (one for each sample and each component) and the red arrows are the loadings, or direction vectors. The direction vectors suggest that placement on the x axis (Comp 1) is largely influenced by body size and % of di-methyl alkanes, while the other covariates determine the y axis (Comp 2) placement of the scores. Without further information, we can use this to suggest that body size and % of di-methyl alkanes oppose each other, so if body size helps survival, then di-methyl alkanes might do the opposite. However, like the quote I mentioned before, there is no guarantee that the dimensionality of these components are associated with the outcome variable (survival). 

Because this is a PLSR, though, we know Comp 1 explains LT50, and we can see the loading weights of each covariate in Comp 1 to claim which covariates most likely positively or negatively influenced survival. Making claims like this is similar to how we interpret the covariates or predictors of a multiple linear regression, so I'll show both the PLSR loading weights and the summary of a multiple regression below (using the basic `lm( )` function in R).

## III: PLSR results and interpretation 

![Table2_LM_PLSR](https://user-images.githubusercontent.com/15988774/209012367-4b0df348-a20c-4a09-b1b5-3c68b9de4f61.jpg)

The results of these two analyses are similar, but the multiple regression can't be safely interpreted. More specifically, its coefficients are ridiculously large, and the % of tri-methyl alkanes can't even be considered because it is so correlated with the other covariates. The fact that we still have significant values is actually a good sign, because usually inflated coefficients also leads to inflated standard errors and overlapping confidence intervals. The PLSR, being a version of PCA, removes all problems of multicollineairity because the covariates are transformed into an orthogonal matrix, and the loading weights of each covariate in each component does not require a p-value to be considered relevant to interpretation. 

For Comp 1, body size has the strongest positive effect and % of di-methyl alkanes has the strongest negative effect. We saw this on the biplot. Since Comp 1 explains the Y variance the best, higher body size is associated with increased survival, while a higher % of di-methyl alkanes is associated with decreased survival. This makes sense when we look at Comp 1 vs. LT50:

![PC1_vs_LT50d](https://user-images.githubusercontent.com/15988774/209012440-acf1150b-83d5-469a-a35f-88e6db3d6dfb.jpg)

So Comp 1 really aligns with survival during desiccation, and we know which covariates define Comp 1. The remaining covariates help explain a different dimension of the data. Instead of correlating well with LT50, Comp 2 seems to separate the northern and southern nests in terms of their remaining CHC properties (% n-alkanes, mono-methyl, tri-methyl, and chain lengths):

![PC2_vs_LT50d](https://user-images.githubusercontent.com/15988774/209012543-c5600233-5e36-477c-98f5-392eac84ced9.jpg)

This is as far as my interpretation and understanding go, right now. Perhaps this new analysis invites new investigations into the CHC profiles. For instance, does it make sense to consider di-methyl alkanes has being worse at water proofing than the other methyl alkanes? Currently, however, the biggest flaw of this method is the use of proportions. When trying to find differences between parameters (e.g. X1 and X2), if we convert them to proportions, then their values become relative to each other, and a smaller/larger X1 could be due to X1 actually being smaller/larger, or simply X2 and other parameters being smaller/larger. So it is still better if we can look at our covariates not as proportions, and not summarized into classes either (which removes/ignores their individual effects). 
***
