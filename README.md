# Genimoic prediction following Clumping and P-value Threasholding using PRSice

## PBL Scenario 
Imagine we are part of a research team studying a quantitative trait (e.g., height). We have GWAS summary statistics from a large external study, and you have genotype + phenotype data for a cohort (lets say target dataset). Now the task is to build a polygenic risk score (PRS) and evaluate how well it predicts the trait in target cohort.

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
In our practice will use following GWAS summary statistics which is GCTA-COJO format (ukb_matched.ma)
```
SNP A1 A2 freq b se p N
rs10000010 C T 0.482739 0.01369 0.03341 0.682 340643
rs1000007 C T 0.278497 -0.008524 0.03699 0.8177 345363
rs10000141 A G 0.090938 0.1598 0.05742 0.005391 347975
rs1000016 G A 0.066432 -0.0627 0.06689 0.3486 341770
rs10000169 C T 0.250075 -0.03871 0.03808 0.3094 348501
rs10000272 C T 0.056848 0.09971 0.07128 0.1618 348501
rs10000282 T C 0.088018 -0.0006234 0.05836 0.9915 347413
rs1000031 A G 0.335671 -0.01188 0.03502 0.7345 346070
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
--base /QRISdata/Q9427/ISG2026/ukb_matched.ma \
--target /QRISdata/Q9427/ISG2026/validation_1kg/1000G_phase3.eur \
--pheno /QRISdata/Q9427/ISG2026/validation_1kg/1kg.eur.simulated.phen \
--beta  \
--pvalue p \
--stat b \
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
--bar-levels 0.001,0.05,0.1,0.2,0.3,0.4,0.5,1 \
--clump-kb 250kb \
--clump-p 1.000000 \
--clump-r2 0.100000
```
## Output
The command will generate following output file 
### output.prsice
```
Pheno   Set     Threshold       R2      P       Coefficient     Standard.Error  Num_SNP
-       Base    0.001   0.460142        5.26825e-69     643.511 31.172  1306
-       Base    0.05    0.359991        1.91153e-50     3223.74 192.23  8813
-       Base    0.1     0.309847        3.26869e-42     4401.03 293.743 14379
-       Base    0.2     0.272322        1.98276e-36     6247.03 456.685 23865
-       Base    0.3     0.25515         7.01947e-34     7688.56 587.485 31775
-       Base    0.4     0.243653        3.31817e-32     8926.01 703.311 38609
-       Base    0.5     0.234101        7.82242e-31     9884.88 799.596 44345
-       Base    1       0.229755        3.25307e-30     13970.2 1143.93 64264
```
### output.summary
```
Phenotype       Set     Threshold       PRS.R2  Full.R2 Null.R2 Prevalence      Coefficient     Standard.Error  P       Num_SNP
-               Base    0.001   0.460142        0.460142        0       -       643.511            31.172  5.26825e-69     1306


```
### output.best
```
FID IID In_Regression PRS
HG00097 HG00097 Yes -0.00209781776
HG00099 HG00099 Yes -0.0078045559
HG00100 HG00100 Yes -0.00516328484
HG00101 HG00101 Yes -0.00330218224
HG00102 HG00102 Yes 0.00306305513
HG00103 HG00103 Yes 0.0024911562
HG00105 HG00105 Yes 0.0062098392
```
### In your working directory you will have output.mismatch and output.log files. Please have a look to these files.
- The ouput.mismatch file lists SNPs removed because their alleles didn’t match between the base GWAS and the target genotype data.
- The output.log file is simply a record of what a program did while it was running. 


## Step4: Evaluate Prediction in R
```
prs <- read.table("output.best", header=TRUE)
pheno <- read.table("1kg.eur.simulated.phen", header=TRUE)
merged <- merge(prs, pheno, by=c("FID","IID"))

summary(lm(pheno ~ PRS, data=merged))

```

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

# Wrap‑Up
- PRSice automates clumping + thresholding.
- R² peaks at the threshold where the PRS includes enough true causal SNPs to capture polygenicity.
- PRS performance depends on genetic architecture.
- PRS is a baseline — not the final model.

### Furher practice task
- Change the clumping and p value threasholding parameters and re‑run PRSice.
- Compare R² and number of SNPs.
- Check prediction improve or worsen and try to interprete.
- Try Binary phenotype.
