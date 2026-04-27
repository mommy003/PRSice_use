# PRSice_use

# PBL Scenario 
Imagine we are part of a research team studying a quantitative trait (e.g., height). We have GWAS summary statistics from a large external study, and you have genotype + phenotype data for a cohort (lets say target dataset). Now the task is to build a polygenic risk score (PRS) and evaluate how well it predicts the trait in your cohort.

### Learning outcome
- We want to predict a phenotype using genome‑wide SNP effects.
- We would like to learn clumping, thresholding, scoring, and model evaluation.”
- We will solve a real problem: build a PRS and evaluate its predictive power.”

### We shold think about 
- What information do we need from the base GWAS?
- Why do we need clumping?
- Why do we test multiple P‑value thresholds?


To download latest version of PRSice please go to https://choishingwan.github.io/PRSice/ and transfer the folder in your working directory
# Step1: Prepare base (reference) 
GWAS summary statistics using suitable software (e.g. –linear function in PLINK). Please note that, PRSice needs a clean GWAS summary file with consistent SNP IDs and effect alleles. Typical columns names required as SNP, A1, A2, BETA/OR, P, N. Compress and index GWAS file if large (e.g., bgzip, tabix) for convenience. 
# Step2: Prepare target genotype and phenotype files
- Target file should be PLINK –bfile (target.bed, target.bim, target.fam). Also note that pgen file can also be used. 
- Phenotype should have three columns (FID IID PHENO)
- Covariate file: FID IID + covariates (e.g., age, sex, PC1–PC10)
- $\color{red}{Important}$ Ensure FID and IID match between PLINK, phenotype, and covariate files
# Step3: Run Basic PRSice command 
```
./PRSice_linux \
--base TOY_BASE_GWAS.assoc \
--target TOY_TARGET_DATA \
--pheno TOY_TARGET_DATA.pheno \
--stat OR \
--bar-levels 1e-8,1e-7,1e-6,1e-5,3e-5,1e-4,3e-4,0.001,0.003,0.01,0.03,0.1,0.3,1 \
--fastscore \
--out test
```
User can include covariate in the model to estimate PGS/PRS by adding following flags
```
--cov covar.txt \
--cov-col age,sex,PC1-PC10 \
```

## Aforementioned basic PRSice vasic (without covariate) command will automatically enable clumping and P value thresholding
- Use PRSice’s built in clumping and multiple thresholds to scan prediction performance.
- Key flags: --clump-kb, --clump-r2, --clump-p, --bar-levels 
- PRSice will compute PRS at each threshold and test association with the phenotype
- For binary traits, it will report Nagelkerke R² / AUC; for quantitative traits, R²

```
 --clump-kb 250kb \
 --clump-p 1.000000 \
 --clump-r2 0.100000
```
# Output
The command will generate three output file 
- test.summary 
- test.best
- test.all.score
