# PRSice_use

# PBL Scenario 
Imagine we are part of a research team studying a quantitative trait (e.g., height). We have GWAS summary statistics from a large external study, and you have genotype + phenotype data for a cohort (lets say target dataset). Now the task is to build a polygenic risk score (PRS) and evaluate how well it predicts the trait in your cohort.

### Learning outcome
- We want to predict a phenotype using genome‑wide SNP effects.
- We would like to learn clumping, thresholding, scoring, and model evaluation.
- We will solve a real problem: build a PRS and evaluate its predictive power.

### We should think about 
- What information do we need from the base GWAS?
- Why do we need clumping?
- Why do we test multiple P‑value thresholds?


## To download latest version of PRSice please go to https://choishingwan.github.io/PRSice/ and transfer the folder in your working directory


## Step1: Prepare base (reference) 
GWAS summary statistics using suitable software (e.g. –linear function in PLINK). Please note that, PRSice needs a clean GWAS summary file with consistent SNP IDs and effect alleles. Typical columns names required as SNP, A1, A2, BETA/OR, P, N. Compress and index GWAS file if large (e.g., bgzip, tabix) for convenience. One of the easiest way to run GWAS using PLINK and PLINK output could easily be used for culping and theasholding. Please see the following output as an example

```
CHR     SNP     BP      A1      TEST    NMISS   BETA    STAT    P
1       rs3934834       995669  T       ADD     2973    0.7677  1.426   0.1541
1       rs3737728       1011278 A       ADD     2998    -0.4979 -1.195  0.2321
1       rs6687776       1020428 T       ADD     2997    0.3382  0.6506  0.5154
1       rs9651273       1021403 A       ADD     3000    -0.1792 -0.4192 0.6751
1       rs4970405       1038818 G       ADD     2996    -0.1893 -0.3049 0.7605
1       rs12726255      1039813 G       ADD     2995    -0.07247        -0.1306 0.8961
1       rs9660710       1089205 A       ADD     2992    0.08227 0.1078  0.9142
1       rs1320565       1109721 T       ADD     2998    0.2502  0.3577  0.7206
1       rs11260549      1111657 A       ADD     3000    0.1459  0.2553  0.7985
1       rs9729550       1125105 C       ADD     2985    -0.547  -1.325  0.1854
```
## Step2: Prepare target genotype and phenotype files
- Target file should be PLINK –bfile (target.bed, target.bim, target.fam). Also note that pgen file can also be used. 
- Phenotype should have three columns (FID IID PHENO)
- Covariate file: FID IID + covariates (e.g., age, sex, PC1–PC10)
- $\color{red}{Important}$ Ensure FID and IID match between PLINK, phenotype, and covariate files
## Step3: Run Basic PRSice command 
```
./PRSice_linux \
--base TOY_BASE_GWAS.assoc \
--target TOY_TARGET_DATA \
--pheno TOY_TARGET_DATA.pheno \
--stat OR \
--fastscore \
--out test
```
User can include covariate in the model to estimate PGS/PRS by adding following flags
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
 --clump-kb 250kb \
 --clump-p 1.000000 \
 --clump-r2 0.100000
```
## Output
The command will generate three output file 
- test.summary 
- test.best
- test.all.score
