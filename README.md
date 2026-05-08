# Genimoic prediction following Clumping and P-value Threasholding using PRSice


### Phenotype simulation
We simulated a quantitative phenotype with a heritability of 0.5 captured by 100 causal variants in 3,000 unrelated UKB participants with an European ancestry using [GCTA](https://yanglab.westlake.edu.cn/software/gcta/#GWASSimulation), and performed GWAS with [PLINK 1.9](https://www.cog-genomics.org/plink/1.9/assoc). Using effects of the 100 causal variants simulated, we calculated the phenotype of the 502 EUR participants of the Phase 3 data in the [1000 Genomes Project](https://www.internationalgenome.org/) as the true phenotypes. Polygenic scores will be generated for the 502 participants using the C + PT method implemented in [PRSice](https://choishingwan.github.io/PRSice/) and the SBayesRC method (https://www.nature.com/articles/s41588-024-01704-y) implemented in [GCTB](https://gctbhub.cloud.edu.au/software/gctb/#SBayesRCTutorial). We will go through the two methods and compare the accuracy achieved by the two methods.  

## PBL Scenario 
Imagine we are part of a research team studying a quantitative trait (e.g., height). We have GWAS summary statistics from a large external study, and we have genotype + phenotype data for a cohort (lets say target dataset). Now the task is to build a polygenic risk score (PRS) and evaluate how well it predicts the trait in target cohort.

### Learning outcome
- Participants will able to predict a phenotype using genome‑wide SNP effects.
- Participants will able to learn clumping, thresholding, scoring, and model evaluation.
- Participants will able to solve a real problem: build a PRS and evaluate its predictive power.

### We should think about 
- What information do we need from the base GWAS?
- Why do we need clumping?
- Why do we test multiple P‑value thresholds?
- How much variance explained (R²)?

### To download latest version of PRSice please go to https://choishingwan.github.io/PRSice/ and transfer the folder in your working directory

## Step1: Prepare base GWAS (Reference) 
GWAS summary statistics using suitable software (e.g. –linear function in $\color{red}{PLINK}$). Please note that, PRSice needs a clean GWAS summary file with consistent SNP IDs and effect alleles. Typical columns names required as $\color{blue}{SNP}$, $\color{blue}{A1}$, $\color{blue}{A2}$, $\color{blue}{BETA/OR}$, $\color{blue}{P}$, $\color{blue}{N}$. Compress and index GWAS file if large (e.g., bgzip, tabix) for convenience. GWAS output from PLINK could easily be used for culmping and theasholding when using PRSice. Please see the following output as an example

```
CHR     SNP              BP     A1      TEST    NMISS   BETA    STAT    P
1       rs3934834       995669  T       ADD     2973    0.7677  1.426   0.1541
1       rs3737728       1011278 A       ADD     2998    -0.4979 -1.195  0.2321
1       rs6687776       1020428 T       ADD     2997    0.3382  0.6506  0.5154
1       rs9651273       1021403 A       ADD     3000    -0.1792 -0.4192 0.6751
1       rs4970405       1038818 G       ADD     2996    -0.1893 -0.3049 0.7605
1       rs12726255      1039813 G       ADD     2995    -0.07247 -0.1306 0.8961
1       rs9660710       1089205 A       ADD     2992    0.08227 0.1078  0.9142
```
In our practice will use following GWAS summary statistics which is GCTA-COJO format (e.g. ukb_matched.ma). Please note that GCTA-COJO format shoul have follow this order ```SNP A1 A2 freq b se p N```

```
SNP A1 A2 freq b se p N
rs10000010 C T 0.477993 -0.2796 0.3719 0.4522 2999
rs10000030 A G 0.135302 -0.09771 0.5496 0.8589 2997
rs1000007 C T 0.275333 -0.1548 0.4153 0.7094 3000
rs10000121 G A 0.468092 -0.5717 0.3729 0.1253 2993
rs10000141 A G 0.087725 0.6495 0.6649 0.3287 2998
rs1000016 G A 0.068402 0.6553 0.7509 0.3829 2997
rs10000169 C T 0.250833 -0.1971 0.4317 0.648 3000
```
## Step2: Prepare target genotype and phenotype files
- Target file should be PLINK –bfile (target.bed, target.bim, target.fam). Also note that pgen file can also be used. 
- Phenotype should have three columns (FID IID PHENO)
- Covariate file: FID IID + covariates (e.g., age, sex, PC1–PC10)
- $\color{red}{Important}$ Ensure FID and IID match between PLINK, phenotype, and covariate files
## Step3: Run Basic PRSice command 
```
./PRSice_linux \
--a1 A1 \
--base /QRISdata/Q9427/ISG2026/gwas.ma \
--target /QRISdata/Q9427/ISG2026/validation_1kg/1000G_phase3.eur \
--pheno /QRISdata/Q9427/ISG2026/validation_1kg/1kg.eur.simulated.phen \
--beta  \
--pvalue p \
--stat b \
--bar-levels 1e-8,1e-7,1e-6,1e-5,3e-5,1e-4,3e-4,0.001,0.003,0.01,0.03,0.1,0.3,1 \
--binary-target F \
--fastscore  \
--out output 
```
User can include covariate in the model to estimate PGS/PRS by adding following flags. In this practice we did not consider covariates.
```
--cov covar.txt \
--cov-col age,sex,PC1-PC10 \
```

## Aforementioned basic PRSice basic (without covariate) command will automatically enable clumping and P value thresholding
- Use PRSice’s built in clumping and multiple thresholds to scan prediction performance.
- Key flags: --clump-kb, --clump-r2, --clump-p, --bar-levels 
- PRSice will compute PRS at each threshold and test association with the phenotype
- For binary traits, it will report Nagelkerke R² / AUC; for quantitative traits, R²

```
--bar-levels 0.001,0.05,0.1,0.2,0.3,0.4,0.5,1 \ #these bar levels are default
--clump-kb 250kb \
--clump-p 1.000000 \
--clump-r2 0.100000
```
## Output
The command will generate following output file 
### output.prsice
```
Pheno   Set     Threshold       R2      P       Coefficient     Standard.Error  Num_SNP
-       Base    1e-08   0.36329 5.21757e-51     13.7373 0.813319        16
-       Base    1e-07   0.399268        2.35661e-57     16.8538 0.924532        21
-       Base    1e-06   0.420875        2.38487e-61     19.3876 1.01707 25
-       Base    1e-05   0.440827        3.58456e-65     22.8621 1.15152 31
-       Base    3e-05   0.428636        8.05541e-63     29.8688 1.54222 43
-       Base    0.0001  0.430858        3.0278e-63      40.0594 2.05903 62
-       Base    0.0003  0.350101        9.00969e-49     59.2422 3.60971 121
-       Base    0.001   0.207611        4.11104e-27     80.7644 7.05633 271
-       Base    0.003   0.135748        1.36819e-17     116.815 13.1816 645
-       Base    0.01    0.0796285       1.20641e-10     167.694 25.4965 1794
-       Base    0.03    0.0520202       2.39184e-07     253.624 48.4193 4859
-       Base    0.1     0.0327873       4.48802e-05     386.427 93.8621 13663
-       Base    0.3     0.0102986       0.0229677       418.955 183.673 32836
-       Base    1       0.00923048      0.0313783       758.556 351.461 68729
```
### output.summary
```
Phenotype       Set     Threshold       PRS.R2  Full.R2 Null.R2 Prevalence      Coefficient     Standard.Error  P       Num_SNP
-       Base    1e-05   0.440827        0.440827        0       -       22.8621 1.15152 3.58456e-65     31
```
### output.best
```
FID IID In_Regression PRS
HG00097 HG00097 Yes 0.415548387
HG00099 HG00099 Yes 0.221354839
HG00100 HG00100 Yes 0.207306452
HG00101 HG00101 Yes 0.471112903
HG00102 HG00102 Yes 0.335629032
HG00103 HG00103 Yes 0.255645161
HG00105 HG00105 Yes 0.229096774
HG00106 HG00106 Yes -0.0472741935
HG00107 HG00107 Yes 0.144548387
HG00108 HG00108 Yes 0.26416129
HG00109 HG00109 Yes 0.320064516
```
### In your working directory you will have output.mismatch and output.log files. Please have a look to these files.
- The ```output.mismatch``` file lists SNPs removed because their alleles didn’t match between the base GWAS and the target genotype data.
- The ```output.log``` file is simply a record of what a program did while it was running. 


## Step4: Evaluate Prediction in R
```
prs <- read.table("output.best", header=TRUE)
pheno <- read.table("1kg.eur.simulated.phen", header=TRUE)
merged <- merge(prs, pheno, by=c("FID","IID"))

summary(lm(pheno ~ PRS, data=merged))

```
## $\color{red}{Concerns}$:  as PRSice eitimates p value threasholding in the target data set. Might be there is an issue of overfitting [Tutorial of PRS](https://www.nature.com/articles/s41596-020-0353-1)
### Would it be expected to estimate R² as 0.46?
Please note that heriatbilty of simulated phenotype was 0.5. As we know from the theory, the upper bound of the R² is the true heritability.

### Would you expect similar R² if you apply p-value thresholding in a another indepentdent dataset?
Think what would be the justification of your answer.

## $\color{green}{Probable}$ $\color{green}{solutions}$ : Estimate best p value thresholding in tuning sample and applied best threshold to the target sample to see how R² is changing

### Run PRSice on tuneing sample (--keep ind_100) to scan threshold and pick best one based on R² 
```
./PRSice_linux \
--a1 A1 \
--base /QRISdata/Q9427/ISG2026/gwas.ma \
--target /QRISdata/Q9427/ISG2026/validation_1kg/1000G_phase3.eur \
--pheno /QRISdata/Q9427/ISG2026/validation_1kg/1kg.eur.simulated.phen \
--keep ind_100 \
--beta  \
--pvalue p \
--stat b \
--bar-levels 1e-8,1e-7,1e-6,1e-5,3e-5,1e-4,3e-4,0.001,0.003,0.01,0.03,0.1,0.3,1 \
--binary-target F \
--fastscore  \
--out tune

```
### Now recompute PRS and R² by appling best p value threshold (from ```tune.prsice```) in target dataset for final evaluation ($\color{green}{out-of-sample}$ $\color{green}{prediction}$)
```
./PRSice_linux \
--a1 A1 \
--base /QRISdata/Q9427/ISG2026/gwas.ma \
--target /QRISdata/Q9427/ISG2026/validation_1kg/1000G_phase3.eur \
--pheno /QRISdata/Q9427/ISG2026/validation_1kg/1kg.eur.simulated.phen \
--keep ind_400 \
--beta  \
--bar-levels 1e-6 \
--pvalue p \
--stat b \
--binary-target F \
--fastscore  \
--out target
```
#### ${\color{blue}Now please compare ```tune.summary``` and ```target.summary```. The R² in the tuning set (0.4927 at p < 1e‑06) decreases to 0.4136 in the target set, reflecting an ~16% reduction in accuracy. Difference is extected to be changed if we use larger sample size.}$ 

# Q & A
### What does clumping do?
Clumping removes SNPs that are in high linkage disequilibrium (LD) with each other, keeping only the most significant SNP in each LD block. So, Clumping keeps independent SNPs and removes correlated ones.

### What does P‑value thresholding do?
Thresholding selects SNPs based on their GWAS P‑value, creating multiple PRS at different significance cutoffs. Thresholding tests how many SNPs to include in the PRS.

### Why does R² peak at a certain threshold?
Because moderate P‑value thresholds include enough true causal SNPs without adding too much noise.
This is the point where signal is maximized relative to noise.

### Why does prediction drop at very loose thresholds?
Loose thresholds include too many non‑associated SNPs, adding noise that dilutes the true genetic signal, causing R² to fall.

### Why does too strict a threshold reduce prediction?
Strict thresholds include only genome‑wide significant SNPs, missing thousands of small‑effect variants.
This leads to underfitting and lower R².

# $\color{darkblue}{Wrap‑Up}$
- PRSice automates clumping + thresholding.
- R² peaks at the threshold where the PRS includes enough true causal SNPs to capture polygenicity.
- PRS performance depends on genetic architecture of a trait.
- PRSice (default flags) is a baseline, not the final model.

### Further practice task
- Change the clumping and p value threasholding parameters and re‑run PRSice.
- Compare R² and number of SNPs.
- Check prediction improve or worsen and try to interprete.
- Try to identify best p value threshold in tuning sample and predict the the phenotype in target dataset.
- Try Binary phenotype.
