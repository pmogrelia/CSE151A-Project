# CSE151A-Project

## Dataset Link (2 pts)
UCI Machine Learning Repository – Detection of IoT Botnet Attacks (N-BaIoT):  
https://archive.ics.uci.edu/dataset/442/detection+of+iot+botnet+attacks+n+baiot

## Environment Setup Requirements
Tested with Python 3.11.

Required packages:
```
pip install pandas numpy seaborn matplotlib scikit-learn patool
```
Also install an external archiver (7-Zip or WinRAR) so `patoolib` can call it.

## Data Exploration
### Observations
Benign traffic loaded from all 9 devices: **555,932 rows**   
Attack traffic rows: **6506674** .  
Total working dataset used so far: **7062606**.

### Columns / Features
Total columns in working benign dataset (after adding labels): 119
- 115 numeric network telemetry feature columns (engineered statistical features over temporal decay windows)
- 3 categorical device column: `Device`, `Attack_Type` and `Botnet_Type`
- 1 binary target column: `is_attack`

So merged frame columns count = 119 (when `is_attack` added).

#### Network Columns Structure 
Prefixes/Feature Types:
- `MI_...` Mutual-information–style directional weights / stats
- `H_...` Host-based aggregate statistics
- `HH_...` Host-host (pairwise) stats
- `HH_jit_...` Jitter-related temporal variability stats
- `HpHp_...` Higher-order / socket or port pair statistics
(5 total)

Each Feature Type/Prefix also has 5 window levels. 
Scale tokens (decay / window levels): `L5`, `L3`, `L1`, `L0.1`, `L0.01` (long → short windows).

For each of these prefix-window level combinations we have statistics that can include SOME of the following.
Stat suffixes: weight, mean, variance, std, magnitude, radius, covariance, pcc (8 total).

5 * 5 * 8 = 200 (However not all are included)

Exceptions are listed here (Generated using an LLM)
MI_dir_: missing std, magnitude, radius, covariance, pcc 
H_: missing std, magnitude, radius, covariance, pcc 
HH_jit_: missing std, magnitude, radius, covariance, pcc 
HH_: missing variance 
HpHp_: missing variance

5*5 + 5*5 + 5*5 + 5 + 5 = 85 <br>
200 - 85 = 115


#### Data Scales & Types
- All 115 feature columns: Continuous numeric (ratio / sometimes approximately positive real-valued). Distributions are typically **right‑skewed** (heavy tails) for weights and magnitudes; means cluster around device‑specific baselines; variances / std features show sparse larger spikes.
- `Device`: Nominal categorical (9 distinct values):
	- Danmini_Doorbell
	- Ecobee_Thermostat
	- Ennio_Doorbell
	- Philips_B120N10_Baby_Monitor
	- Provision_PT_737E_Security_Camera
	- Provision_PT_838_Security_Camera
	- Samsung_SNH_1011_N_Webcam
	- SimpleHome_XCS7_1002_WHT_Security_Camera
	- SimpleHome_XCS7_1003_WHT_Security_Camera
- is_attack: Binary (0 = benign, 1 = attack).
- Attack_Type: Nominal categorical (3 distinct values)
    - udp.csv
    - udpplain.csv
    - scan.csv
    - ack.csv
    - syn.csv
    - junk.csv
    - combo.csv
    - tcp.csv
- Botnet_Type: Nominal categorical (3 distinct values)
    - Gafgyt
    - Mirai

### Target Column
is_attack (binary):
- 0 → benign traffic
- 1 → attack traffic 
  
### Missing Values
Computed in notebook: **0 missing values** across all loaded columns.

### Pre-processing data plan

- Remove highly correlated features to avoid redundancy and multicollinearity
- Handle outliers so that extreme values do not dominate training
- Drop duplicates and unneeded columns to keep the dataset clean and efficient
- Balance the data by equally sampling between:
  - attack vs benign traffic
  - different botnet families
  - different attack types
- Normalize and standardize features so that all variables contribute equally to the model
- Remove data with minimal weight (very small subsets or irrelevant samples) to reduce noise

### Duplicate Rows
Computed in notebook: **157779** across entire df. 

## Data Plots
