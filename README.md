Peer Tutoring and Mental Health  
Replication Package

### 1. Overview

This folder contains the code and cleaned datasets used to reproduce the empirical results for the paper *“Peer Tutoring and Mental Health”* based on the China Education Panel Survey (CEPS).  
All descriptions below are based on the provided Stata `.do` files, Python `.py` scripts, and the output tables in the `results` folder.

The replication workflow has two main stages:
- **Data preparation (Python)**: construct analysis datasets from CEPS raw data and generate cleaned `.dta` files.
- **Estimation and presentation (Stata)**: use the cleaned `.dta` files to run regressions and produce the tables and figures reported in the paper.

### 2. Software and user‑written commands

- **Stata**
  - The Stata code uses the following user‑written commands:
    - `reghdfe`
    - `outreg2`
    - `estpost` / `esttab`
    - `coefplot`
  - The master Stata script turns `pause on` and `set more off` to allow non‑interactive execution.

- **Python**
  - The Python scripts are written for Python 3 and use the following libraries (as seen in the `.py` files):
    - `pandas`
    - `numpy`
    - `scikit-learn` (`PCA`, `StandardScaler`)
    - `scipy` (`scipy.stats.ks_2samp` in the balance tests)
    - `pandas.api.types.CategoricalDtype`
    - Standard modules such as `io`, `sys`, and `contextlib.redirect_stdout`

Users should ensure all required Stata packages and Python libraries are installed before running the replication.

### 3. Folder structure and main files

The replication materials are organized as follows (paths are relative to this `replication` folder):
- **`raw data/`** (expected, not distributed here)
  - The Python scripts refer to the following CEPS baseline and wave‑2 files (file names taken from the code):
    - Baseline data (`raw data/basline data/`):
      - `Parent data.dta`  
      - `Student data.dta`  
      - `Teacher data.dta`  
      - `Principal data.dta`
    - Wave‑2 data (`raw data/wave2 data/`):
      - `cepsw2studentCN.dta`  
      - `cepsw2parentCN.dta`  
      - `cepsw2teacherCN.dta`  
      - `cepsw2principalCN.dta`
  - These files correspond to the CEPS baseline (2013–2014) and follow‑up (2014–2015) datasets documented by the China Social Survey Data Archive (CNSDA); see the [CEPS baseline project page](https://www.cnsda.org/index.php?r=projects/view&id=72810330) and the [CEPS follow‑up project page](https://www.cnsda.org/index.php?r=projects/view&id=61662993) for information on data access and questionnaires.

- **`cleaned data/`** – cleaned datasets used for regression analysis
  - `Main_data.dta` – main analysis dataset used by `Main_Tables.do` and `Figures.do`.  
  - `Appendix_data.dta` – robustness analysis dataset used by `Appendix_Tables.do` and `Figures.do`.  
  - `Appendix_table_12_data.dta` – dataset for Appendix Table 12 descriptive statistics.  
  - `Table_R1_data_jde_random.dta` – two‑wave dataset for Table R1 under the JDE random assignment definition.  
  - `Table_R1_data_our_random.dta` – two‑wave dataset for Table R1 under this paper’s random assignment definition.  
  - `KS_balance_data.dta` – dataset including KS‑based class‑pair imbalance measures used in balance‑sample robustness checks.

- **`pyfiles/`** – Python data‑processing scripts
  - `Master.py`  
    - Defines `DATA_PATH` as the `pyfiles` directory and provides a helper function `run_script`.  
    - Sequentially executes:
      - `Main_Tables_Data.py`  
      - `Appendix_Tables_Data.py`  
      - `Balance_Test_Data.py`  
      - `Table_R1_Data.py`  
    - Each script is run via `exec` and is responsible for preparing a specific cleaned dataset or set of outputs.
  - `Main_Tables_Data.py`  
    - Processes CEPS baseline (“Parent”, “Student”, “Teacher”, “Principal”) data.  
    - Merges student, parent, class (homeroom teacher), and school (principal) data.  
    - Defines random class assignment (`random`), keeps only random‑assignment samples, and constructs:
      - Mental health indices (PCA‑based and alternative constructions).  
      - Individual, family, and class‑level control variables.  
      - Time‑use variables (sleep, homework, cramwork, cram_time, exercise, reading, TV, games, housework, entertainment share).  
      - Class atmosphere and friendship variables.  
      - Peer variables based on the 10 closest classmates in terms of academic performance and robustness variants.  
    - Saves the final main analysis dataset as `cleaned data/Main_data.dta`.
  - `Appendix_Tables_Data.py`  
    - Processes the same CEPS baseline data to construct variables needed for robustness checks in the appendix.  
    - Reuses helper functions from `Main_Tables_Data.py`.  
    - Defines different peer groups (e.g. top 8, 10, 12 closest peers) and alternative peer tuition statistics (quantiles, standard deviation).  
    - Computes class‑level peer means excluding the individual student.  
    - Constructs inverse‑covariance mental health indices used in robustness checks.  
    - Saves the resulting dataset as `cleaned data/Appendix_data.dta`.
  - `Balance_Test_Data.py`  
    - Merges baseline student, parent, class, and school data.  
    - Defines random and strict random class assignment indicators.  
    - Constructs tutoring expenditure, mental health indices, and control variables.  
    - Restricts to random‑assignment samples with exactly two classes per school‑grade.  
    - Performs Kolmogorov–Smirnov balance tests across class pairs for key variables (mental health, tutoring, demographics, and family background).  
    - Saves the balance‑test summary table (corresponding to Table 2 in the paper) to the `results` folder and writes the KS‑based class‑pair imbalance measure (`sig_diff_count`) to `cleaned data/KS_balance_data.dta` for use in Appendix robustness checks.
  - `Table_R1_Data.py`  
    - Processes CEPS baseline and wave 2 data.  
    - Defines two random assignment standards:
      - JDE random assignment (`jde_random`).  
      - This paper’s random assignment (`our_random`).  
    - For each standard in turn:
      - Restricts to the corresponding random‑assignment sample.  
      - Imputes economic status and other controls.  
      - Constructs wave‑1 and wave‑2 test scores and tutoring expenditure.  
      - Merges baseline and follow‑up data (keeping successfully followed‑up students without re‑assignment).  
      - Constructs mental health measures at wave 2.  
      - Defines several peer tutoring measures:
        - Based on grade‑6 rank (`peer_rank6_tuition`),  
        - Based on baseline (wave‑1) scores (`peer_w1_tuition`),  
        - Based on the average of wave‑1 and wave‑2 scores (`peer_w12_tuition`).  
      - Saves:
        - `cleaned data/Table_R1_data_jde_random.dta` for the JDE standard.  
        - `cleaned data/Table_R1_data_our_random.dta` for this paper’s standard.

- **`dofiles/`** – Stata scripts for generating tables and figures
  - `Master.do`  
    - Sets the global path to the `dofiles` folder.  
    - Sequentially runs:
      - `Main_Tables.do` (main text Tables 1–14 and related analyses),
      - `Appendix_Tables.do` (Appendix Tables 1–11),
      - `Appendix_Table_12.do` (Appendix Table 12 descriptive statistics),
      - `Table_R1.do` (Table R1, two‑wave robustness table),
      - `Figures.do` (Appendix Figures).
  - `Main_Tables.do`  
    - Uses the cleaned dataset `cleaned data/Main_data.dta`.  
    - Changes the working directory to `results`.  
    - Generates:
      - **Table 1** – Summary statistics (`Table_1.rtf`), with individual‑level summary statistics of key variables.  
      - **Table 2** – Balancing test on paired classrooms: generated externally by `Balance_Test_Data.py` and referenced in the comments as `Table_2.rtf`.  
      - **Table 3** – Balancing test at the individual level (`Table_3.rtf`), using `reghdfe` with individual and family covariates.  
      - **Table 4** – Baseline regression (`Table_4.rtf`), estimating the effect of peer tutoring expenditure on the principal mental health index with progressively richer controls.  
      - **Table 5** – Academic competition / peer substitution mechanism (`Table_5.rtf`).  
      - **Table 6** – Time allocation mechanism (`Table_6.rtf`), with outcomes such as tutoring time, homework, sleep, and leisure time.  
      - **Table 7** – Class atmosphere mechanism (`Table_7.rtf`), using standardized indices of class atmosphere and friendship composition.  
      - **Table 8** – Alternative explanation: economic conditions (`Table_8.rtf`).  
      - **Table 9** – Alternative explanation: academic performance (`Table_9.rtf`).  
      - **Table 10** – Alternative explanation: teaching mode (`Table_10.rtf`).  
      - **Table 11** – Alternative explanation: parental pressure spillovers (`Table_11.rtf`).  
      - **Table 12** – Subgroup analysis (`Table_12.rtf`).  
      - **Tables 13A–13E** – Further analysis on the effects of cram schooling and related outcomes (`Table_13A.rtf`–`Table_13E.rtf`).  
      - **Tables 14A–14E** – Further analysis using percentile rank outcomes (`Table_14A.rtf`–`Table_14E.rtf`).
  - `Appendix_Tables.do`  
    - Uses the cleaned dataset `cleaned data/Appendix_data.dta`.  
    - Changes the working directory to `results`.  
    - Generates Appendix robustness tables:
      - **Appendix Table 1** – Different mental health measures (`Appendix Table 1.rtf`).  
      - **Appendix Table 2** – Different mental health index construction methods (`Appendix Table 2.rtf`).  
      - **Appendix Table 3** – Different peer definitions (`Appendix Table 3.rtf`).  
      - **Appendix Table 4** – Different peer tuition measures (`Appendix Table 4.rtf`).  
      - **Appendix Table 5** – Alternative cram school measures (`Appendix Table 5.rtf`).  
      - **Appendix Table 6** – Excluding small classes (`Appendix Table 6.rtf`).  
      - **Appendix Table 7** – Strict balance sample, using the KS‑based imbalance measure (`Appendix Table 7.rtf`).  
      - **Appendix Table 8** – Excluding grade 9 (`Appendix Table 8.rtf`).  
      - **Appendix Table 9** – Defining peers by predetermined academic performance (`Appendix Table 9.rtf`).  
      - **Appendix Table 10** – Including class mean gender (`Appendix Table 10.rtf`).  
      - **Appendix Table 11** – Different clustering levels (`Appendix Table 11.rtf`).
  - `Appendix_Table_12.do`  
    - Uses the dataset `cleaned data/Appendix_table_12_data.dta`.  
    - Changes the working directory to `results`.  
    - Generates descriptive statistics for Appendix Table 12:
      - `Appendix_table_12_full_sample_individual_stats.rtf` – full sample individual characteristics.  
      - `Appendix_table_12__full_sample_class_stats.rtf` – full sample class characteristics.  
      - `Appendix_table_12_full_sample_school_stats.rtf` – full sample school characteristics.  
      - `Appendix_table_12_random_individual_stats.rtf` – random‑assignment sample individual characteristics.  
      - `Appendix_table_12_random_class_stats.rtf` – random‑assignment sample class characteristics.  
      - `Appendix_table_12_random_school_stats.rtf` – random‑assignment sample school characteristics.
  - `Table_R1.do`  
    - Uses the two‑wave cleaned datasets `cleaned data/Table_R1_data_jde_random.dta` and `cleaned data/Table_R1_data_our_random.dta`.  
    - Changes the working directory to `results`.  
    - Generates:
      - **Table R1A** (`Table R1A.rtf`) – based on the JDE random assignment standard.  
      - **Table R1B** (`Table R1B.rtf`) – based on this paper’s random assignment standard.
  - `Figures.do`  
    - Uses `cleaned data/Main_data.dta` and `cleaned data/Appendix_data.dta`.  
    - Generates:
      - **Appendix Figure 1** – histogram of the PCA mental health index.  
      - **Appendix Figure 2** – coefficient plot comparing the baseline PCA index with alternative PCA indices that exclude one item at a time.

- **`results/`** – output folder (not included in the replication package)
  - When you run the Python and Stata scripts, all regression tables, descriptive‑statistics tables, and figure files are automatically saved here (for example, the main text tables, appendix tables, Table R1, and the appendix figures).

### 4. Key constructed variables (from code)

This section summarizes key variables constructed in the Python and Stata code, using the meanings documented in code comments.

- **Mental health measures**
  - Five underlying questionnaire items are converted to numeric scales (1–5) and stored as:
    - `blue`, `depressed`, `unhappy`, `not_enjoying_life`, `sad`.  
  - Main PCA‑based indices:
    - `pca_mental` / `pca` – principal component index of poor mental health constructed from the five items.  
  - Alternative indices in baseline data:
    - `mental_total` – standardized sum of the five standardized items.  
    - `mental_total_std` – standardized version of `mental_total`.  
    - `mental_JDE` – simple (unweighted) average of the five items.  
    - `Inverse_covariance` – index constructed using inverse‑covariance weights of the (standardized) mental health items.  
  - Exclusion‑based indices (sensitivity checks):
    - `mental_exc_blue`, `mental_exc_depressed`, `mental_exc_unhappy`, `mental_exc_not_enjoy`, `mental_exc_sad` – PCA indices each dropping one of the five items in turn.  
  - Wave‑2 mental health (two‑wave data):
    - `mental_JDE_w2` – simple average of five wave‑2 mental health items.  
    - `pca_mental_w2` – PCA‑based mental health index at wave 2.

- **Tutoring variables**
  - `tuition` – extracurricular tutoring expenditure; derived from relevant questionnaire items, with non‑logical answers set to missing, top‑coded at 10,000, and then converted to thousand yuan.  
  - `cram_p` – indicator for any positive tutoring expenditure.  
  - `cram_time` – weekly tutoring time, constructed from weekday and weekend tutoring hours and minutes, converted to hours and capped at 28 hours per week.

- **Time‑use variables**
  - Sleep:
    - `sleep_time` – daily sleep hours (capped between 4 and 12 hours).  
    - `sleep_week` – weekly sleep hours (`sleep_time * 7`).  
  - Study and homework:
    - `homework` – weekly time spent on school homework, in hours (capped at 35).  
    - `cramwork` – weekly time spent on homework assigned by parents or tutoring centers, in hours (capped at 28).  
  - Tutoring:
    - `cram_time` – weekly time spent in cram schools (see above).  
  - Leisure and housework:
    - `exercise` – weekly time spent on sports and exercise (capped at 21).  
    - `book` – weekly time spent reading books (capped at 21).  
    - `tv` – weekly time spent watching TV (capped at 21).  
    - `game` – weekly time spent using the internet or playing games (capped at 21).  
    - `housework` – weekly time spent doing housework (capped at 21).  
    - `fun_time` – sum of `exercise`, `game`, `tv`, and `book`.  
    - `fun_ratio` – share of entertainment time in total time, defined as  
      `fun_ratio = fun_time / (fun_time + homework + housework + cramwork)`.

- **Individual and family characteristics**
  - `male` – indicator for male student.  
  - `czhk` – indicator for non‑agricultural household registration.  
  - `liushou` – indicator for left‑behind child (both parents not at home).  
  - `only` – indicator for only child.  
  - `dibao`, `water`, `toilet`, `desk` – indicators for components of household economic status.  
  - `eco` – economic status index constructed from `dibao`, `water`, `toilet`, and `desk` (via PCA).  
  - `medu`, `fedu` – years of education of mother and father, mapped from categorical schooling levels.

- **Class and school characteristics**
  - `tmale` – indicator for male homeroom teacher.  
  - `ttitle` – homeroom teacher professional title (collapsed to “junior”, “intermediate”, “senior” categories).  
  - `subject` – subject taught by homeroom teacher (e.g. Chinese, math, English, other).  
  - `texperience` – homeroom teacher’s teaching experience.  
  - `G9` – dummy for grade 9 (grade 7 is the omitted category).  
  - `clsn` – class size.  
  - `best_sch` – indicator for the best school in the district.  
  - `public_sch` – indicator for public school.  
  - `p_shifan` – indicator that the principal graduated from a normal school.  
  - `t_shifan` – indicator that the homeroom teacher graduated from a normal school.  
  - Class‑level averages (examples):  
    - `mean_male`, `cls_male` – share of male students in class.  
    - `cls_czhk`, `cls_liushou`, `cls_only`, `cls_tuition`, `cls_medu`, `cls_fedu`, `cls_eco` – class‑level averages of relevant variables.

- **Random assignment indicators**
  - Baseline / main‑tables data:
    - `random` – indicator that both principal and homeroom teacher report random class assignment according to a set of questionnaire conditions.  
    - `strict_random` – stricter indicator requiring consistent homeroom teacher responses within a school‑grade and additional principal conditions.  
  - Two‑wave data:
    - `jde_random` – random assignment indicator following the JDE standard.  
    - `our_random` – random assignment indicator following this paper’s stricter standard.

- **Peer variables (baseline and appendix data)**
  - The main peer groups are defined as classmates whose academic performance (test score) is closest to the index student’s score, within the same class.
  - Baseline definitions:
    - `peer_tuition` – average tutoring expenditure of the 10 closest classmates by score.  
    - `peer_score` – average score of the same peer group.  
    - `peer_male`, `peer_only`, `peer_liushou`, `peer_czhk`, `peer_eco`, `peer_medu`, `peer_fedu` – average peer characteristics for the same peer group.  
  - Robustness variants:
    - `peer_8_tuition`, `peer_12_tuition` and corresponding peer characteristics based on 8 or 12 closest classmates.  
    - `peer_q25_tuition`, `peer_q50_tuition` (median), `peer_q75_tuition`, `peer_std_tuition` – peer tuition based on different statistics (quartiles and standard deviation).  
    - `peer_iqr_tuition` – interquartile range (Q75 – Q25) of peer tuition.  
    - `peer_robust_tuition` and corresponding `peer_robust_*` characteristics – averages for the 10 classmates with the largest score differences (robustness check).  
    - `peer_class_tuition` and `peer_class_*` – class‑level peer means excluding the individual student.

- **Peer variables (two‑wave data, Table R1)**
  - `peer_rank6_tuition` and corresponding peer characteristics – defined based on grade‑6 rank.  
  - `peer_w1_tuition` and `peer_w1_*` – peers defined by baseline (wave‑1) scores.  
  - `peer_w12_tuition` and `peer_w12_*` – peers defined by the average of wave‑1 and wave‑2 scores.

- **Balance‑test variables**
  - In `Balance_Test_Data.py`, the following are used for KS balance tests across class pairs (two classes within the same school‑grade):
    - `pca_mental`, `tuition`, `male`, `czhk`, `only`, `liushou`, `medu`, `fedu`, `eco`.  
  - `sig_diff_count` – for each school‑grade class pair, the number of variables with significantly different distributions (p‑value ≤ 0.05). This measure is later merged into `KS_balance_data.dta` and used in Appendix Table 7.

### 5. Mapping from scripts to tables and figures

Below is a script‑to‑output mapping based on in‑file comments and table labels in the paper:

- **Python (`pyfiles/`)**
  - `Main_Tables_Data.py` → prepares `cleaned data/Main_data.dta` for the main text tables and figures.  
  - `Appendix_Tables_Data.py` → prepares `cleaned data/Appendix_data.dta` for appendix robustness tables and figures.  
  - `Balance_Test_Data.py` → prepares `cleaned data/KS_balance_data.dta` and generates the balance‑test table (Table 2) saved in `results`.  
  - `Table_R1_Data.py` → prepares `cleaned data/Table_R1_data_jde_random.dta` and `cleaned data/Table_R1_data_our_random.dta` for Table R1.

- **Stata main tables (`dofiles/Main_Tables.do`)**
  - Produces all main‑text tables (Tables 1–14), including summary statistics, individual‑level balance tests, baseline regressions, mechanisms (time allocation and class atmosphere), alternative explanations, subgroup analysis, and further analysis of cram schooling and academic performance. All corresponding tables are saved to the `results` folder.

- **Stata appendix tables (`dofiles/Appendix_Tables.do`)**
  - Produces Appendix Tables 1–11, covering robustness checks with different mental health measures, index construction methods, peer definitions and peer tuition measures, alternative cram school measures, exclusion of small classes and grade 9, alternative peer definitions based on predetermined performance, inclusion of class mean gender, and alternative clustering levels. All tables are saved to the `results` folder.

- **Stata Appendix Table 12 (`dofiles/Appendix_Table_12.do`)**
  - Produces the individual‑, class‑, and school‑level descriptive statistics underlying Appendix Table 12 for both the full sample and the random‑assignment subsample, saved as tables in the `results` folder.

- **Stata Table R1 (`dofiles/Table_R1.do`)**
  - Produces the two‑wave robustness table (Table R1), with one panel based on the JDE random assignment standard and one panel based on this paper’s random assignment standard, saved to the `results` folder.

- **Stata figures (`dofiles/Figures.do`)**
  - Produces Appendix Figure 1 (histogram of the PCA mental health index) and Appendix Figure 2 (coefficient plot comparing baseline and exclusion‑based mental health indices), saving the figures to the `results` folder.

### 6. How to run the replication

This section describes how to reproduce the cleaned datasets and all tables and figures using the provided scripts. Paths in the code are hard‑coded to the author’s machine but are marked with comments such as “Change the path to your local machine if necessary”.

1. **Prepare CEPS raw data**
   - Obtain access to the CEPS baseline and wave‑2 datasets.  
   - Create the following subfolders under `replication` (if they do not already exist), matching the paths used in the Python scripts:
     - `raw data/basline data/`  
     - `raw data/wave2 data/`
   - Place the CEPS `.dta` files in these folders with the filenames expected by the scripts (as listed in Section 3).

2. **Adjust file paths if necessary**
   - In the Python scripts (`Main_Tables_Data.py`, `Appendix_Tables_Data.py`, `Balance_Test_Data.py`, `Table_R1_Data.py`) and in the Stata scripts (`Master.do`, `Main_Tables.do`, `Appendix_Tables.do`, `Appendix_Table_12.do`, `Table_R1.do`, `Figures.do`), file paths are specified using absolute Windows paths on the author’s computer.  
   - Follow the in‑file comment “Change the path to your local machine if necessary” and update:
     - `DATA_PATH`, `RAW_DATA_PATH`, `BASELINE_DATA_PATH`, `WAVE2_DATA_PATH`, `CLEANED_DATA_PATH` in the Python scripts so that they point to your local `replication` directory.  
     - The `global PATH` and any `use` / `cd` paths in the Stata `.do` files so that they match your local folder structure.

3. **Generate cleaned datasets and balance‑test table (Python)**
   - In a Python 3 environment with the required libraries installed, set the working directory to `replication/pyfiles`.  
   - Run:
     - `Master.py`  
   - This script will execute, in order:
     - `Main_Tables_Data.py` → creates `cleaned data/Main_data.dta`.  
     - `Appendix_Tables_Data.py` → creates `cleaned data/Appendix_data.dta`.  
     - `Balance_Test_Data.py` → creates `cleaned data/KS_balance_data.dta` and writes `results/Table_2.rtf`.  
     - `Table_R1_Data.py` → creates `cleaned data/Table_R1_data_jde_random.dta` and `cleaned data/Table_R1_data_our_random.dta`.

4. **Generate main and appendix tables and figures (Stata)**
   - Open Stata and set the working directory to `replication/dofiles` (or any directory from which paths in the `.do` files are valid).  
   - Ensure that `reghdfe`, `outreg2`, `estpost`/`esttab`, and `coefplot` are installed.  
   - Run:
     - `Master.do`  
   - `Master.do` will then run, in sequence:
     - `Main_Tables.do` – generates all main‑text tables (Tables 1–14) and saves them to the `results` folder.  
     - `Appendix_Tables.do` – generates Appendix Tables 1–11 and saves them to the `results` folder.  
     - `Appendix_Table_12.do` – generates the Appendix Table 12 descriptive‑statistics tables in the `results` folder.  
     - `Table_R1.do` – generates the Table R1 results in the `results` folder.  
     - `Figures.do` – generates the appendix figures (saved according to the figure options specified in the `.do` file) in the `results` folder.

5. **Check outputs**
   - All regression and descriptive‑statistics tables, as well as the appendix figures, are written into the `results` folder.  
   - The cleaned datasets in `cleaned data/` correspond to the datasets referenced in the Stata scripts, and no additional manual data manipulation is required beyond the scripted steps.

### 7. Data access and confidentiality

All analyses in this replication package use data from the China Education Panel Survey (CEPS).  
The replication materials do **not** redistribute CEPS data; CEPS baseline (2013–2014) and follow‑up (2014–2015) data, together with their questionnaires and documentation, are available from the China Social Survey Data Archive (CNSDA) at the CEPS baseline project page and the CEPS follow‑up project page cited above.  
Given access to the CEPS data, the Python and Stata scripts in this folder reproduce the cleaned datasets, tables, and figures described in this README.

