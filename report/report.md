---
title: 'Larkin Ricker in Salmon population '
author: "Rui Zhang"
output:
  pdf_document:
    fig_caption: yes
    number_sections: yes
    toc: yes
  html_document:
    fig_caption: yes
    number_sections: yes
    theme: flatly
    toc: yes
    toc_depth: 2
csl: ecology.csl

---




---

<big><big><big>Objective<big><big><big>

* Finding a model for cyclic dominance in sockeye salmon population.

* Statistically determining whether the Larkin Ricker model is needed.

<br>

---

# Introduction

It is a common fact that the population of salmon  flutuates periodically, with a cohort that hugely outnumbers the others. This phenomenon is usually called cyclic dominance. Some researches have been done and claim that cyclic dominance is possibly caused by environmental variablities when the population of fish is in some kinds of situations, such as narrow spawning age distribution and low persistence (White et al.[1], Botsford et al [2]).

The populations of the sockeye salmons in Fraser River periodically flutuates with period four, which is the same as generation time. A cycle with period four can have many appearance. Except for random evironmental noise, is there any further mechanism that cause this pattern -- cyclic dominance? My goal is to analyze the interaction, if any, between cohorts.

# Data

My data is from the supplemental material of White el al.[1]. We can take a glance at the Seymour data, which is one of the 19 Fraser River sockeye stocks. 


```r
library(RCurl)
Url=getURL("https://raw.githubusercontent.com/ruizhang-ray/salmon_larkin/master/data/Seymour.csv")
seymour=read.table(text=Url,header = FALSE,sep=',')
colnames(seymour)=c("year","eff","rec3","rec4","rec5")
summary(seymour)
```

```
##       year           eff              rec3              rec4       
##  Min.   :1948   Min.   :   311   Min.   :    0.0   Min.   :  1944  
##  1st Qu.:1963   1st Qu.:  2869   1st Qu.:    0.0   1st Qu.: 18347  
##  Median :1978   Median :  6585   Median :  136.0   Median : 55150  
##  Mean   :1978   Mean   : 14854   Mean   :  911.6   Mean   :124433  
##  3rd Qu.:1994   3rd Qu.: 18984   3rd Qu.:  564.0   3rd Qu.:175000  
##  Max.   :2009   Max.   :108000   Max.   :23882.0   Max.   :812000  
##                                  NA's   :3         NA's   :4       
##       rec5      
##  Min.   :    0  
##  1st Qu.:  191  
##  Median : 1934  
##  Mean   : 5083  
##  3rd Qu.: 7081  
##  Max.   :42537  
##  NA's   :5
```

We can see there are about 62-year data, and in each year there are effective female spawner (eff) and adult recruits with age three, four and five (rec3, rec4, rec5).

According to the run-timing, the 8 most abundant sockeye stocks can be divided into four groups (Peterman et al[4]). Followings are the plots of Early Stuart, Seymour and Late Stuart, which return before all other Fraser stocks, at early-summer and at mid-summer, respectively. 


```r
Url=getURL("https://raw.githubusercontent.com/ruizhang-ray/salmon_larkin/master/data/Seymour.csv")
seymour=read.table(text=Url,header = FALSE,sep=',')
row.aval=which(!is.nan(apply(seymour,1,sum)))
seymour=as.matrix(seymour[row.aval,])
seymour=cbind(seymour,apply(seymour[,3:5],1,sum))
for (i in 1:3){
  seymour=cbind(seymour,seymour[,2+i]/seymour[,6])
}
colnames(seymour)=c('year','eff','rec3','rec4','rec5','rec.total','r3.per','r4.per','r5.per')

Url=getURL("https://raw.githubusercontent.com/ruizhang-ray/salmon_larkin/master/data/EarlyStuart.csv")
earlystuart=read.table(text=Url,header = FALSE,sep=',')
row.aval=which(!is.nan(apply(earlystuart,1,sum)))
earlystuart=as.matrix(earlystuart[row.aval,])
earlystuart=cbind(earlystuart,apply(earlystuart[,3:5],1,sum))
for (i in 1:3){
  earlystuart=cbind(earlystuart,earlystuart[,2+i]/earlystuart[,6])
}
colnames(earlystuart)=c('year','eff','rec3','rec4','rec5','rec.total','r3.per','r4.per','r5.per')

Url=getURL("https://raw.githubusercontent.com/ruizhang-ray/salmon_larkin/master/data/LateStuart.csv")
latestuart=read.table(text=Url,header = FALSE,sep=',')
row.aval=which(!is.nan(apply(latestuart,1,sum)))
latestuart=as.matrix(latestuart[row.aval,])
latestuart=cbind(latestuart,apply(latestuart[,3:5],1,sum))
for (i in 1:3){
  latestuart=cbind(latestuart,latestuart[,2+i]/latestuart[,6])
}
colnames(latestuart)=c('year','eff','rec3','rec4','rec5','rec.total','r3.per','r4.per','r5.per')


###-------data display--------

##Seymour
par(mfrow=c(1,1))
plot(seymour[,'year'],seymour[,'rec4']/10,type='b',col='red',main='Seymour',xlab='year',ylab='population')
lines(seymour[,'year'],seymour[,'rec3'],type='b',col='blue')
lines(seymour[,'year'],seymour[,'rec5'],type='b',col='green')
lines(seymour[,'year'],seymour[,'rec.total']/10,type='b',col='black')
legend(seymour[1,'year'],70000,c('rec3','rec4','rec5','rec.total'),col=c('blue','red','green','black'),lty=c(1,1,1))
```

<img src="figure/plots_of_3_stocks-1.pdf" title="plot of chunk plots_of_3_stocks" alt="plot of chunk plots_of_3_stocks" style="display: block; margin: auto;" />

```r
max(seymour[,'r3.per'])
```

```
## [1] 0.05174147
```

```r
mean(seymour[,'r3.per'])
```

```
## [1] 0.005044865
```

```r
max(seymour[,'r5.per'])
```

```
## [1] 0.4846439
```

```r
mean(seymour[,'r5.per'])
```

```
## [1] 0.07176551
```

```r
## EarlyStuart
par(mfrow=c(1,1))
plot(earlystuart[,'year'],earlystuart[,'rec4']/10,type='b',col='red',main='EarlyStuart',xlab='year',ylab='population')
lines(earlystuart[,'year'],earlystuart[,'rec3'],type='b',col='blue')
lines(earlystuart[,'year'],earlystuart[,'rec5'],type='b',col='green')
lines(earlystuart[,'year'],earlystuart[,'rec.total']/10,type='b',col='black')
legend(earlystuart[1,'year'],170000,c('rec3','rec4','rec5','rec.total'),col=c('blue','red','green','black'),lty=c(1,1,1))
```

<img src="figure/plots_of_3_stocks-2.pdf" title="plot of chunk plots_of_3_stocks" alt="plot of chunk plots_of_3_stocks" style="display: block; margin: auto;" />

```r
max(earlystuart[,'r3.per'])
```

```
## [1] 0.02394665
```

```r
mean(earlystuart[,'r3.per'])
```

```
## [1] 0.001962232
```

```r
max(earlystuart[,'r5.per'])
```

```
## [1] 0.4022141
```

```r
mean(earlystuart[,'r5.per'])
```

```
## [1] 0.08478021
```

```r
## LateStuart
par(mfrow=c(1,1))
plot(latestuart[,'year'],latestuart[,'rec4']/10,type='b',col='red',main='LateStuart',xlab='year',ylab='population')
lines(latestuart[,'year'],latestuart[,'rec3'],type='b',col='blue')
lines(latestuart[,'year'],latestuart[,'rec5'],type='b',col='green')
lines(latestuart[,'year'],latestuart[,'rec.total']/10,type='b',col='black')
legend(latestuart[1,'year'],470000,c('rec3','rec4','rec5','rec.total'),col=c('blue','red','green','black'),lty=c(1,1,1))
```

<img src="figure/plots_of_3_stocks-3.pdf" title="plot of chunk plots_of_3_stocks" alt="plot of chunk plots_of_3_stocks" style="display: block; margin: auto;" />

```r
max(latestuart[,'r3.per'])
```

```
## [1] 0.02164611
```

```r
mean(latestuart[,'r3.per'])
```

```
## [1] 0.00384186
```

```r
max(latestuart[,'r5.per'])
```

```
## [1] 0.7077784
```

```r
mean(latestuart[,'r5.per'])
```

```
## [1] 0.1019538
```

1. To make the plots comfortable to be read, the number of four-year-old adult returns and the total number of the adult returns are divided by 10.

2. The populations of these stocks are mainly composed by the four-year-old adult returns since the trajectory of the red dots and black dots are very the same.

3. From the plot of Seymour stocks, we can see the five-year-old adult returns are abundant right after the years of the appearances of dominant cohorts. This is predictable because both of them are the off-springs of the last dominant cohorts. However, from the plots of Early Stuart and Late Stuart, we can see most of the five-year-old adult returns are abundant at the same year as the abundance of whole population. There are two possible accounts for this. First, the number of five-year-old adult returns are miscounted. As we can see, the total population of the fish and the percentage of five-year-old adult returns of Early Stuart and Late Stuart are higher than the Seymour. Is there any chance that people miscount the large four-year-old fish as the five-year-old fish and this error become severer when the whole population is larger? Second, it is also possible that the shared living environment during growth of the fish that return at the same year cause this simultaneous abundance of them. 

4. We can notice that in 1967, the donminant cohort of Seymour delayed one year, in 1996 the dominant cohorts in both Early Stuart and Late Stuart shifted to one year early. Phase switching and shifting are common in the population that flutuates periodically (White et al.[1]). An example of phase switching in cyclic population is pink salmon (Krkosed et al.[7])

Furthermore, we can investigate the correlation plots.


```r
##cross correlation of rec4 & rec5
ccf(seymour[,'rec4'],seymour[,'rec5'])
```

<img src="figure/correlations_of_3_stock-1.pdf" title="plot of chunk correlations_of_3_stock" alt="plot of chunk correlations_of_3_stock" style="display: block; margin: auto;" />

```r
ccf(earlystuart[,'rec4'],earlystuart[,'rec5'])
```

<img src="figure/correlations_of_3_stock-2.pdf" title="plot of chunk correlations_of_3_stock" alt="plot of chunk correlations_of_3_stock" style="display: block; margin: auto;" />

```r
ccf(latestuart[,'rec4'],latestuart[,'rec5'])
```

<img src="figure/correlations_of_3_stock-3.pdf" title="plot of chunk correlations_of_3_stock" alt="plot of chunk correlations_of_3_stock" style="display: block; margin: auto;" />

```r
# evidences that there is strong positive correlation  between the recuirts with same
# returning year instead of the same birth year
```

1. In these three stocks, the populations of five-year-old adult returns and four-year-old adult returns at the same years are highly positive correlated.  

2. In the plots of Early Stuart and Late Stuart, we can see the population of  five-year-old adult returns are highly positive correlated with the population of four-year-old adult returns at the four years before.


# Model
Process model:
$$\begin{aligned}
F_t=\alpha (R_{t,4}+R_{t,5})exp(&\gamma_0 R_{t,4}+\omega_0 R_{t,5}+\\
&\gamma_1 R_{t-1,4}+\omega_1 R_{t-1,5}+\\
&\gamma_2 R_{t-2,4}+\omega_2 R_{t-2,5}+\\
&\gamma_3 R_{t-3,4}+\omega_3 R_{t-3,5}+\epsilon_t)\hspace{0.5in}[Larkin Ricker]\\
\end{aligned}
$$
$$\begin{aligned}
\epsilon_t&\sim N(0,\sigma^2)\\
p_t&=1/(1+exp(a+bR_{t,4}+cR_{t,5}))\\
X_{t,4}&=p_tF_t\\
X_{t,5}&=(1-p_t)F_t\\
R_{t,4}&=X_{t-4,4}\\
R_{t,5}&=sX_{t-5,5}\\
\end{aligned}
$$

Measurement model:
$$\begin{aligned}
Rec_4\sim LogNormal(log(\phi_1 R_{t,4}),\psi_1)\\
Rec_5\sim LogNormal(log(\phi_2 R_{t,5}),\psi_2)\\
\end{aligned}
$$

1. In order to reduce the redundancy, effective female spawner (eff) is not included. Moreover, the three-year-old adult return (rec3) is not included as well because of its small amount.

2. $F_t$ is the newborn generation which is given birth at year $t$. $R_{t,k}$ is the $k$-year-old adult return that return spawning ground at year $t$. $X_{t,k}$ is part of $F_t$, which will return spawning ground $k$ years later. $s$ is the survival rate from four years old to five years old.

3. $p_t$ is the proportion that $X_{t,4}$ takes among $F_t$. Mathematically, it is a logistic regression, of which the outcome is between 0 to 1. Biologically, it is assumed that the proportion that the fry will return four years later is related to the number of their parents of different ages.

4. $\alpha$ is often referred as productive rate (Peterman et al.[6], Myers et al.[5]). However here, it is the product of productive rate and the accumulative survival rate till four years old.

5. This is a state space model with non-linearity, which makes the iterated filtering  a good algorithm for analysis.
#Model comparison and analysis

In this part, a few tests will be done to analyze the significance of the parameters. Furthermore, the Larkin Ricker model will be compared with the standard Ricker model, Beveton-holt model, Beveton-holt model with delayed density dependence (Larkin version) and AMAR model.

Beveton-holt with delayed density dependence:
$$\begin{aligned}
F_t=\frac{\alpha (R_{t,4}+R_{t,5})}{\gamma_0 R_{t,4}+\gamma_1 R_{t-1,4}+\gamma_2 R_{t-2,4} +\gamma_3 R_{t-3,4}+\omega_0 R_{t,5}+\omega_1 R_{t-1,5}+\omega_2 R_{t-2,5}+\omega_3 R_{t-3,5}}exp(\epsilon_t)
\end{aligned}
$$

Figure 1 is the results (b:Beveton-holt):

![AIC Table](AIC.png)

1. Model 1 is the model of Null Hypothesis that the number of five-year-old adult returns are not significant and only two-year delayed density dependence is needed.  

2. By comparing 2 with 1, we can know that the number of five-year-old adult returns are not significant for computing the proportion of four-year-old adult return.

3. By comparing 3 with 1, we can know that the delayed density dependence on five-year-old adult returns are not significant.

4. We can see all the Ricker and Beveton-holt model outperform the ARMA model. Choosing by AIC, the best model is model 11, which includes three-year delay density dependence. This model has the log likelihood much higher than the others and is the only model that the Beveton-holt version are not close to the Ricker version.

Following is the R codes that building pomp object for the model 11:


```r
library(RCurl)
Url=getURL("https://raw.githubusercontent.com/ruizhang-ray/salmon_larkin/master/data/Seymour.csv")
seymour=read.table(text=Url,header = FALSE,sep=',')
row.aval=which(!is.nan(apply(seymour,1,sum)))
seymour=as.matrix(seymour[row.aval,])
seymour=cbind(seymour,apply(seymour[,3:5],1,sum))
for (i in 1:3){
  seymour=cbind(seymour,seymour[,2+i]/seymour[,6])
}
colnames(seymour)=c('year','eff','rec3','rec4','rec5','rec.total','r3.per','r4.per','r5.per')
##-------pomp--------
library(pomp)
library(ggplot2)
data=data.frame(seymour)
##normalizing

#data$eff=data$eff/sd(data$eff)
data$rec3=data$rec3/max(data$rec.total)
data$rec4=data$rec4/max(data$rec.total)
data$rec5=data$rec5/max(data$rec.total)
data$rec.total=data$rec.total/max(data$rec.total)
```

Notice that I normalize adult returns data by dividing them by the max total adult return.


```r
salmon_statenames <-c("F0","F1","F2","F3","F4","F5",
                      "R04","R14","R24","R34","R44","R54",
                      "R05","R15","R25","R35","R45","R55")
salmon_rp_names <- c("alpha",
                     "gamma0","gamma1","gamma2","gamma3",
                     "a","b","c","s5","sigma","phi1","phi2","psi1","psi2")
salmon_ivp_names <- c("F0_0","F1_0","F2_0","F3_0","F4_0","F5_0",
                      "R04_0","R14_0","R24_0","R34_0","R44_0","R54_0",
                      "R05_0","R15_0","R25_0","R35_0","R45_0","R55_0")
salmon_paramnames <- c(salmon_rp_names, salmon_ivp_names)
salmon_rproc <-'
double  epsilon=rnorm(0,sigma);
double  y=0;
double  X4=0;
double  X5=0;

F5=F4;
F4=F3;
F3=F2;
F2=F1;
F1=F0;

R54=R44;
R44=R34;
R34=R24;
R24=R14;
R14=R04;


R55=R45;
R45=R35;
R35=R25;
R25=R15;
R15=R05;

X4=(1/(1+exp(a+b*R44+c*R45)))*F4;
X5=(1-1/(1+exp(a+b*R54+c*R55)))*F5;

R04=X4;
R05=s5*X5;
F0=alpha*(R04+R05)*exp(gamma0*R04+gamma1*R14+gamma2*R24+gamma3*R34+epsilon);
'
salmon_initializer <-"
R04=R04_0;
R14=R14_0;
R24=R24_0;
R34=R34_0;
R44=R44_0;
R54=R54_0;
R05=R05_0;
R15=R15_0;
R25=R25_0;
R35=R35_0;
R45=R45_0;
R55=R55_0;
F0=F0_0;
F1=F1_0;
F2=F2_0;
F3=F3_0;
F4=F4_0;
F5=F5_0;
"
salmon_rmeasure <-"
rec4=rlnorm(log(phi1*R04),psi1);
rec5=rlnorm(log(phi2*R05),psi2);
"
salmon_dmeasure <-'
double tol = 0.00001;
lik=dlnorm(rec4+tol,log(phi1*R04+tol),psi1,1);
lik+= dlnorm(rec5+tol,log(phi2*R05+tol),psi2,1);
if (!((lik>-999)&(lik<999))) lik=-9999;
lik=(give_log) ? lik : exp(lik);
'



salmon_toEst <-"
Tsigma=log(sigma);
Ts5=logit(s5);
Talpha=log(alpha);
TF0_0=log(F0_0);
TF1_0=log(F1_0);
TF2_0=log(F2_0);
TF3_0=log(F3_0);
TF4_0=log(F4_0);
TF5_0=log(F5_0);
Tphi1=log(phi1);
Tphi2=log(phi2);
Tpsi1=log(psi1);
Tpsi2=log(psi2);  

"
salmon_fromEst <-"
Tsigma=exp(sigma);
Ts5=expit(s5);
Talpha=exp(alpha);
TF0_0=exp(F0_0);
TF1_0=exp(F1_0);
TF2_0=exp(F2_0);
TF3_0=exp(F3_0);
TF4_0=exp(F4_0);
TF5_0=exp(F5_0);
Tphi1=exp(phi1);
Tphi2=exp(phi2);
Tpsi1=exp(psi1);
Tpsi2=exp(psi2);

"

n=dim(data)[1]
salmon<- pomp(data=data.frame(rec4=data$rec4[6:n]+data$rec3[6:n],
                              rec5=data$rec5[6:n],year=data[6:n,'year']),
              statenames=salmon_statenames,
              paramnames=salmon_paramnames,
              times="year",
              t0=data$year[5],
              rmeasure=Csnippet(salmon_rmeasure),
              dmeasure=Csnippet(salmon_dmeasure),
              rprocess=discrete.time.sim(step.fun=Csnippet(salmon_rproc),
                                         delta.t=1),
              initializer=Csnippet(salmon_initializer),
              toEstimationScale=Csnippet(salmon_toEst),
              fromEstimationScale=Csnippet(salmon_fromEst)
)
salmon_ivp_fix=c(0,0,data$rec5[1:5],0,data$rec3[1:5]+data$rec4[1:5])
names(salmon_ivp_fix)=c("F5_0","R55_0","R45_0","R35_0","R25_0","R15_0","R05_0",
                         "R54_0","R44_0","R34_0","R24_0","R14_0","R04_0")
```

We can use iterated filtering to search the MLEs. Following is the example code with parallel computing.


```r
run_level=3
CORES<-10 ##update for flux
salmon_Np=c(100,5000,60000)
salmon_Nmif=c(10,200,350)
salmon_Nglobal=c(10,20,120)
salmon_Neval=c(5,20,120)
JOBS=salmon_Nglobal[run_level]
nodefile <- Sys.getenv("PBS_NODEFILE")
CLUSTER <- nchar(nodefile)>0
CLUSTER
if (CLUSTER) {
  require(doSNOW)
  hostlist <- tail(scan(nodefile,what=character(0)),-1)
  cl <- makeCluster(hostlist,type='SOCK')
  cl
  registerDoSNOW(cl)
} else {
  require(doParallel)
  registerDoParallel(CORES)
}
library(pomp)
tic <- Sys.time()
#cat("session1")
mpar <- foreach(
  i=1:JOBS,
  .packages=c('pomp'),
  .inorder=FALSE,.combine=c) %dopar% {
    Sys.sleep(i*.1)
    setwd("~/Larkin/June_27/model11") 
    NMIF<-salmon_Nmif[run_level]
    NP<-salmon_Np[run_level]
    source("salmon.r")
    salmon_box <-rbind(
      alpha=c(0.4,6),
      gamma0=c(-4,4),
      gamma1=c(-4,4),
      gamma2=c(-4,4),
      gamma3=c(-4,4),
      a=c(-6,-2),
      b=c(-8,-3),
      s5=c(0.2,0.6),
      sigma=c(0.05,0.8),
      phi1=c(0.8,1.6),
      phi2=c(0.2,3),
      psi1=c(0.55,0.7),
      psi2=c(1.8,1.9),
      F4_0=c(0.01,0.06),
      F3_0=c(0.01,0.22),
      F2_0=c(0.4,1),
      F1_0=c(0.2,1),
      F0_0=c(0.04,0.08)
    )
    salmon_fix=c(salmon_ivp_fix,c=0)
    salmon.rw.sd <- c(rep(0.02,13),rep(0.1,5))
    names(salmon.rw.sd) <- row.names(salmon_box)
    salmon_cooling.fraction.50 <- 0.5
    
    set.seed(1129+i)
    Sys.sleep(i*0.1)
    th.draw=c(apply(salmon_box,1,function(x) runif(1,x[1],x[2])),salmon_fix)
    cat("session2")
    m<-try(mif2(salmon,
                start=th.draw,
                Np=NP,
                Nmif=NMIF,
                cooling.type="geometric",
                cooling.fraction.50=salmon_cooling.fraction.50,
                transform=TRUE,
                rw.sd=salmon.rw.sd
    ))
    m
  }  
L.if <- foreach(i=1:JOBS,.packages='pomp',
                .combine=rbind,.inorder=FALSE) %dopar% 
{
  Sys.sleep(i*.1)
  setwd("~/Larkin/June_27/model11")
  source("salmon.r")
  set.seed(1993+i)
  Sys.sleep(i*.1)
  logmeanexp(
    replicate(salmon_Neval[run_level],
              logLik(pfilter(salmon,params=coef(mpar[[i]]),Np=salmon_Np[run_level]))
    ),
    se=TRUE)
}
setwd("~/Larkin/June_27/model11") ##turn on for flux
source("salmon.r")

toc <- Sys.time()
print(toc-tic)

r.if=data.frame(logLik=L.if[,1],logLik_se=L.if[,2],t(sapply(mpar,coef)))
summary(r.if$logLik,digits=5)
#18 parameters
pairs(~logLik+s5+alpha+gamma0+gamma1+gamma2+gamma3,
      data=subset(r.if,logLik>max(logLik)-20))
pairs(~logLik+a+b+c,data=subset(r.if,logLik>max(logLik)-20))
pairs(~logLik+alpha+gamma0+gamma1+gamma2+gamma3+sigma+phi1+phi2+psi1+psi2+s5,
      data=subset(r.if,logLik>max(logLik)-20))
plot(mpar)

if (run_level>1) 
  write.table(r.if,file="salmon_global_model11_params.csv",
              append=TRUE,col.names=TRUE,row.names=FALSE,sep=',')
save.image(file="result.rda")
if (CLUSTER) stopCluster(cl)
```

Results are as follows:


```r
load("result.rda")
plot(mpar)
```

<img src="figure/result-1.pdf" title="plot of chunk result" alt="plot of chunk result" style="display: block; margin: auto;" /><img src="figure/result-2.pdf" title="plot of chunk result" alt="plot of chunk result" style="display: block; margin: auto;" /><img src="figure/result-3.pdf" title="plot of chunk result" alt="plot of chunk result" style="display: block; margin: auto;" /><img src="figure/result-4.pdf" title="plot of chunk result" alt="plot of chunk result" style="display: block; margin: auto;" /><img src="figure/result-5.pdf" title="plot of chunk result" alt="plot of chunk result" style="display: block; margin: auto;" />

```r
pairs(~logLik+alpha+gamma0+gamma1+gamma2+gamma3+sigma+phi1+phi2+psi1+psi2+s5,
      data=subset(r.if,logLik>max(logLik)-20))
```

<img src="figure/result-6.pdf" title="plot of chunk result" alt="plot of chunk result" style="display: block; margin: auto;" />
------------

# References

1. _Stochastic models reveal conditions for cyclic dominance in sockeye salmon populations_, J. Wilson White et al.

2. _Cohort resonance: a significant component of fluctuations in recruitment, egg production, and catch of fished populations_, Louis W. Botsfrd et al.

3. _Equation-free mechanistic ecosystem forecasting using empirical dynamic modeling_, Hao Ye et al.

4. _Patterns of covariation in survival rates of British Columbian and Alaskan sockeye salmon (Oncorhynchus nerka) stocks_, Randall M. Peterman et al.

5. _Simple dynamics underlie sockeye salmon (Oncorhynchus nerka) cycles_, Ransom A. Myers et al.

6. _A widespread decrease in productivity of sockeye salmon (Oncorhynchus nerka) populations in western North America_, Randall M. Peterman et al.

7. _Cycles, stochasticity and density dependence in pink salmon population dynamics_, Martin Krkosek et al.
