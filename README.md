# **CNVstat**

# **CNVstat v1.0 : Statistical Association Analysis of Copy Number Variants (CNVs)**

Copy number variants (CNVs) and single nucleotide polymorphisms (SNPs) co-exist throughout the human genome and jointly contribute to phenotypic variations. Thus, it is desirable to consider both types of variants, as characterized by allele-specific copy numbers (ASCNs), in association studies of complex human diseases. Current SNP genotyping technologies capture the CNV and SNP information simultaneously via fuorescent intensity measurements. The common practice of calling ASCNs from the intensity measurements and then using the ASCN calls in downstream association analysis has important limitations. First, the association tests are prone to false-positive findings when differential measurement errors between cases and controls arise from differences in DNA quality or handling. Second, the uncertainties in the ASCN calls are ignored.

**CNVstat** is a command-line program written in C for the statistical association analysis of CNVs and SNPs. **CNVstat** allows the user to estimate or test the effects of CNVs and SNPs by maximizing the (observed-data) likelihood that properly accounts for differential measurement errors and calling uncertainties. It is versatile in several aspects: (1) it provides the integrated analysis of CNVs and SNPs as well as the analysis of total CNVs; (2) it can accommodate both Affymetrix and Illumina data, as well as all platforms that assay CNVs quantitatively, such as array CGH; (3) it accounts for the case-control sampling, differential measurement errors and calling uncertainties; (4) it can be readily extended to other study designs and traits; (5) it formulates the effects of CNVs and SNPs on the phenotype through flexible regression models, which can accommodate various genetic mechanisms and gene-environment interactions; and (6) it allows genetic and environmental variables to be correlated. The program is fast and scalable to genomewide association scans. For example, it took about 2 hrs on a 64-bit, 3.0-GHz Intel Xeon machine to perform the analysis on chromosome 1 of the schizophrenia data (Hu et al. Submitted for publication). We are working intensely to improve the capabilities of **CNVstat**, so please check back frequently for updates.

## **SYNOPSIS**

**CNVstat** [**–spec_file** specification file]

## **OPTIONS**

| Option         | Parameter            | Default               | Description                    |
|:---------------|:---------------------|:----------------------|:-------------------------------|
| **–spec_file** | {specification file} | *`specification.txt`* | Specify the specification file |

## **INPUT FILES**

All the input files are **required**. The current version does\
**not allow** any missing data.

### **specification file**

|                                                     |
|-----------------------------------------------------|
| Example of a specification file                     |
| PHENOTYPE_FILE = ./posted/example_phenotype.dat     |
| RAW_INTENSITY_DAT_FILE = ./posted/example_raw.dat   |
| RAW_INTENSITY_INFO_FILE = ./posted/example_raw.info |
| CALLED_CN_DAT_FILE = ./posted/example_CN.dat        |
| CALLED_CN_INFO_FILE = ./posted/example_CN.info      |
| RESULT_FILE = ./example_results.out                 |
|                                                     |
| DESIGN = case-control                               |
| ARRAY_TYPE = I                                      |
| PROBE_TYPE = SNP                                    |
| EFFECT_MODE = additive                              |
| DIFFERENTIAL_MEASUREMENT_ERRORS = 1                 |
|                                                     |
| CATEGORICAL = sex                                   |
| DEPENDENT = sex                                     |
| EFFECT = K L sex sex\*K sex\*L                      |
| TEST = K L sex\*K sex\*L                            |

The specification file describes parameters that detemine the desired analysis. All the parameters are **optional** and the **default** values are listed below. The syntax follows

> KEYWORD = value1 [value2 …]

with spaces around “=”. An empty value, i.e., “KEYWORD =”, is **not allowed**. Arbitrary empty lines are allowed.

`PHENOTYPE_FILE =`{phenotype file}

> **Must** appear before the parameters `CATEGORICAL`, `DEPENDENT`, `EFFECT` and `TEST`. **Default** is “./phenotype.dat“. More details of the phenotype file and other input files are given below.

`RAW_INTENSITY_DAT_FILE =`{data file of raw intensities}

> **Default** is “./raw.dat“.

`RAW_INTENSITY_INFO_FILE =`{information file of raw intensities}

> **Default** is “./raw.info“.

`CALLED_CN_DAT_FILE =`{data file of called CNs}

> **Default** is “./CN.dat“.

`CALLED_CN_INFO_FILE =`{information file of called CNs}

> **Default** is “./CN.info“.

`RESULT_FILE =`{result file}

> **Default** is “./results.out“.

`DESIGN =`{case-control/cohort/cross-sectional}

> Specify the study design.\
> **Default** is “case-control“. It **must** be “case-control” in the current version.

`ARRAY_TYPE =`{I/A/T}

> Specify the array type among ‘I‘, ‘A‘ and ‘T‘, corresponding to Illumina arrays, Affymetrix arrays, and other arrays that assay CNVs quantitatively, such as array CGH. Specifically, Illumina arrays may contain both SNP and CN probes; a SNP probe generates a measurement for the total copy number, Log R Ratio (LRR), and a measurement for the allelic contrast, B Allele Frequency (BAF), while a CN probe generates only one meausrement for the total copy number. Affymetrix arrays may also contain both SNP and CN probes; a SNP probe generates two measurements pertaining to A and B allele intensities while a CN probe generates one meausrement for the total copy number. ‘T‘ refers to arrays with all probes generating measurements for the total copy numbers.\
> **Default** is ‘I‘.

`PROBE_TYPE =`{SNP/CN}

> Specify the probe type between “SNP” and “CN“, indicating SNP or CN probes when `ARRAY_TYPE` = I/A. Note that when `ARRAY_TYPE` = T, `PROBE_TYPE` **must** be “CN“. **Default** is “SNP“. The current version does **not allow** `PROBE_TYPE` = SNP when `ARRAY_TYPE` = A.

`EFFECT_MODE =`{additive}

> Specify the mode of effect of the copy number changes. It **must** be “additive” in the current version, indicating that the total copy number and the B allele copy number, if specified in `EFFECF`, are included in the disease risk model as linear terms. **Default** is “additive“.

`DIFFERENTIAL_MEASUREMENT_ERRORS =`{1/0}

> Turn on the switch by ‘1‘ to allow differential measurement errors; turn it off by ‘0‘. **Default** is “1“.

`CATEGORICAL =`{covariate names in the phenotype file}

> Specify covariates that are categorical. A categorical covariate is internally transformed into (level-1) indicators with the lowest level as the reference. For example, if smoke has values 1, 2, and 3, it will be transformed into two indicators I(smoke=2) and I(smoke=3) with names “smoke(2)” and “smoke(3)”. Unspecified covariates are assumed to be continuous by **default**.

`DEPENDENT =`{covariate names in the phenotype file}

> Specify covariates that are potentially correlated with genetic variants. It only **allows** one covariate in the current version. Unspecified covariates are assumed to be independent of genetic variants by **default**.

`EFFECT =`{main genetic and environmental effects and interactions}

> Specify the main genetic and environmental effects and interactions in the disease risk model. In particular, the effect of the total copy number is denoted by ‘K‘ and the effect of the B allele copy number is denoted by ‘L‘. Interactions are denoted by ‘\*’ with no space on either side. The interation ‘K\*L‘ or ‘L\*K‘ is **not allowed**. Effect ‘L‘ is **not allowed** when `PROBE_TYPE` = CN. **Default** is ‘K‘.

`TEST =`{main genetic and environmental effects and interactions}

> Specify the main genetic and environmental effects and interactions that are included in the joint test. These effects should be a subset of those specified in `EFFECT`. **Default** is ‘K‘.

### **Phenotype file**

|              |                             |
|--------------|-----------------------------|
|              | Example of a phenotype file |
|           CC |  sex                        |
|            1 |   2                         |
|            1 |   1                         |
|            1 |   2                         |
|            1 |   1                         |
|            0 |   1                         |
|            0 |   2                         |
|            … |   …                         |

The phenotype file provides information on the phenotype and covariates of the study subjects in a tabular (row-column) format. Each row contains space- or tab-delimited data specific to an individual. Variable names **must** be specified in the first line. The disease variable **must** be listed first and can be followed by an arbitrary number of covariates (or no covariate). In a case-control study, the disease variable **must** be coded 0/1 to represent unaffected/affected.

### **Data file of raw intensities**

|                                                            |
|------------------------------------------------------------|
| Example of a data file of raw intensities                  |
| `0.2563 -0.1172 -0.0032  0.5917  0.0460 -0.2151  0.1422 …` |
| `0.4865  0.0341  0.9929  1.0000  0.4787  1.0000  1.0000 …` |
| `0.1170 -0.3365  0.2157  0.7017  0.3450 -0.2712  0.2081 …` |
| `0.4388  0.9548  0.0000  0.0000  0.5589  0.0000  0.0000 …` |
| `…       …       …       …       …       …       …   …`    |

The data file of raw intensities contains raw intensities in a tabular format without any header. Each colomn represents an individual. When `PROBE_TYPE` = SNP, each probe contributes two rows in the order of LRR and BAF if `ARRAY_TYPE` = I or A and B allele intensities if `ARRAY_TYPE` = A. When `PROBE_TYPE` = CN, each probe contributes one row of the intensity for the total CN. The probes are not necessisarily ordered by physical positions on the chromosomes.

### **Information file of raw intensities**

|                                                   |
|---------------------------------------------------|
| Example of an information file of raw intensities |
| `SNP_A-8363651      13      68146300      0.7143` |
| `SNP_A-8363651      13      68146300      0.7143` |
| `SNP_A-8510099      13      68150996      0.2667` |
| `SNP_A-8510099      13      68150996      0.2667` |
| `…            …          …            …`          |

The information file of raw intensities provides information on the probes in a tabular format without any header. The columns are ordered as *probe id*, *chromosome*, *physical position* and *population frequency of the B allele (pfb)*. The rows precisely correspond to the rows of the data file of raw intensities; in particular, a SNP probe contributes two rows while a CN probe contributes one.

### **Data file of called CNs**

|                                               |
|-----------------------------------------------|
| Example of a data file of called CNs          |
| `2     2     2     2     2     2     2     …` |
| `1     1     2     2     2     2     2     …` |
| `…     …     …     …     …     …     …     …` |

The data file of called CNs contains inferred total copy numbers on the study subjects, which can be obtained by a calling algorithm such as PennCNV, to be used for constructing initial values of the parameters of the implemented methods. This file is in a tabular format without any header, with columns and rows corresponding to individuals and probes in the same order as in the data file of raw intensities. Note that a SNP probe only contributes one row. The current version is focused on common CNVs and CNV deletions; that is, only the loci with CN deletions as the possible variants and with the frequency of any deletion exceeding 5% among the study subjects, as estimated by the called CNs, are retained in the input files.

### **Information file of called CNs**

|                                                |
|------------------------------------------------|
| `SNP_A-8363651     13     68146300     0.7143` |
| `SNP_A-8510099     13     68150996     0.2667` |
| `…             …         …           …`        |

: Example of an information file of called CNs   

The information file of called CNs **must** follow the same format and the same order as the information file of raw intensities, except that a SNP contributes one row.

## **RESULTS**

### **Result file**

|                                                                |
|----------------------------------------------------------------|
| `Chr   Position     CN/SNP-id      pfb   Test-Stat   p-Value`  |
| `---  ---------  ---------------  -----  ---------   --------` |
| `13   68146300  SNP_A-8363651    0.714     0.2309   8.91e-01`  |
| `13   68150996  SNP_A-8510099    0.267     0.2739   8.72e-01`  |
| `…       …            …            …         …         …`      |

Example of a result file                                       

`p-Value` is obtained by comparing `Test-Stat` to a Chi-Square distribution with degrees of freedom the number of effects in the joint test.

## **EXAMPLE**

Enter the command

> \$ CNVstat --spec_file ./example_specification.txt

to obtain the results in “example_results.out“.

## **REFERENCE**

Hu, Y. J., Lin, D. Y. Sun, W., and Zeng, D. “A Likelihood-Based Framework for Association Analysis of Allele-Specific Copy Numbers”. *Submitted for publication*.

## **DOWNLOAD**

#### **CNVstat for x86-based, 64-bit Linux [updated July 5, 2012]**

executable (zip archive) **»** [CNVstat-1.0-linux-64.zip](http://dlin.web.unc.edu/wp-content/uploads/sites/1568/2012/07/CNVstat-1.0-linux-64.zip)

#### **Example files [updated July 5, 2012]**

zip archive **»** [CNVstat-1.0-example.zip](http://dlin.web.unc.edu/wp-content/uploads/sites/1568/2012/07/CNVstat-1.0-example.zip)

## **VERSION HISTORY**

| Version | Date      | Description             |
|:--------|:----------|:------------------------|
| 1.0     | Jul. 2012 | First version released. |
