# CSE151A-Project
Link to our [notebook](CSE151A_Project.ipynb)
## Dataset Link 
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

### Duplicate Rows
Computed in notebook: **157779** across entire df. 

## Pre-processing Data Plan

- Remove highly correlated features to avoid redundancy and multicollinearity
- Handle outliers so that extreme values do not dominate training
- Drop duplicates and unneeded columns to keep the dataset clean and efficient
- Balance the data by equally sampling between:
  - attack vs benign traffic
  - different botnet families
  - different attack types
- Normalize and standardize features so that all variables contribute equally to the model
- Remove data with minimal weight (very small subsets or irrelevant samples) to reduce noise

## Data Plots

![piecharts](images/pie.png)

Benign vs Attack Distribution<br>
This plot shows a clear imbalance in the dataset, where attack traffic dominates with about 92.1% of the total data compared to only 7.9% benign traffic. Such imbalance indicates that any model trained on this dataset must account for the disproportionate number of attack samples, otherwise it may simply learn to predict “attack” most of the time.

Botnet_Type Distribution <br>
The dataset consists of two main botnet families, Mirai and Gafgyt (BASHLITE). The plot reveals that Mirai accounts for 56.4% of the attack data, while Gafgyt contributes 43.6%. This relatively even split ensures that both families are well represented, allowing for fairer analysis of how models detect different attack origins.

Attack_Type Distribution for Mirai <br>
Breaking down Mirai further, the plot highlights that UDP-based floods are the most dominant attack type (33.5%), followed by SYN floods (20.0%), ACK floods (17.6%), and smaller proportions of scanning (14.7%) and udpplain attacks (14.3%). This shows that Mirai primarily relies on high-volume flooding techniques, with UDP traffic playing the most significant role.

Attack_Type Distribution for BASHLITE <br>
In contrast, the BASHLITE (Gafgyt) botnet has a more diverse distribution of attack types. The plot indicates that UDP floods (33.3%) and TCP floods (30.3%) are the largest contributors, while combo (18.2%), junk (9.2%), and scanning (9.0%) attacks make up the remainder. Unlike Mirai, BASHLITE employs both UDP and TCP traffic heavily, demonstrating a broader attack strategy.

![correlation heatmap](images/output.png)

Heatmap <br>
The correlation heatmap visualizes the relationships between all numerical features in the dataset. Many features show strong positive correlations, forming clusters that suggest redundancy. This means that some features may not add unique information and could potentially be removed or reduced through dimensionality reduction techniques like PCA. Addressing this correlation structure is important to improve computational efficiency and reduce overfitting during model training.


# Milestone 3

- 1: Finish major preprocessing, which includes scaling and/or transforming your data, imputing your data, encoding your data, and Feature expansion (example is taking features and generating new features by transforming via polynomial, log multiplication of features).  (10 points)
In the preprocessing stage, we performed a series of steps to prepare the dataset for modeling. We began by removing duplicate rows and creating an 80/20 train–test split on the balanced working set. Since the dataset was highly imbalanced, we applied undersampling of the majority class whenever the ratio between classes exceeded 3:1, ensuring a 1:1 distribution of benign and attack samples. All numeric features were scaled using Min–Max scaling, with constant columns handled by setting them to zero. The categorical feature Device was excluded from the feature set to avoid label leakage, though future iterations could include it with proper encoding. These steps ensured that the dataset of over seven million rows was standardized and balanced, providing a strong foundation for training the first model.

- 2: Train your first model and analyze your model's performance. Evaluate your model and compare training vs. test error. (10 points)
For our first model, we trained a Decision Tree classifier with carefully chosen hyperparameters, including a maximum depth of six, a minimum of one thousand samples to split an internal node, five hundred samples per leaf, balanced class weights, and a square-root feature selection strategy. On the held-out test set, the model achieved excellent results: an accuracy of 99.96%, precision of 99.95%, recall of 99.96%, and an F1-score of 99.96. The confusion matrix confirmed these results, with only 46 false positives and 40 false negatives out of over 205,000 test samples. These outcomes show that the decision tree successfully captured the decision boundaries in the data while maintaining a balanced trade-off between precision and recall.

- 3: Answer the questions: Where does your model fit in the fitting graph? (Build at least one model with different hyperparameters and check for over-/underfitting, pick the best model.) What are the next models you are thinking of and why? (5 points)
When evaluating our models against the bias–variance trade-off, we found that our current approach is positioned high on the overfitting side of the fitting graph. The decision tree we trained at a depth of six showed excellent results on the training data, but its very high accuracy on the test set also indicates that the model may be capturing dataset-specific patterns that will not generalize well to truly unseen data. Deeper trees exaggerate this tendency, while shallower trees underfit by failing to capture enough structure. To address this concern, we are considering Support Vector Machines (SVMs) as our next model. SVMs are well-suited for high-dimensional spaces, can provide more robust decision boundaries, and with the right kernel choice and regularization, they can reduce overfitting by controlling the margin and focusing on support vectors rather than memorizing the data. Exploring SVMs with different kernels and hyperparameters will help us test whether a more balanced fit can be achieved.

- 4: Conclusion section: What is the conclusion of your 1st model? What can be done to improve it? (5 points)
Our first model achieved extremely high precision, recall, and F1-score, but its placement high on the overfitting curve signals that caution is needed. While the decision tree demonstrated that the data is highly separable, this also means that the model may not generalize well outside the sampled test set. To mitigate overfitting and improve robustness, our next steps include training Support Vector Machines, which are known for handling high-dimensional and imbalanced data more effectively when tuned with proper kernels and regularization. In addition, we plan to refine preprocessing by encoding categorical features, experimenting with feature transformations, and conducting thorough cross-validation. These adjustments will provide a clearer picture of how to move from a potentially overfit model toward one that balances accuracy and generalization.

## Final Written Report

# Introduction to Your Project
Why was it chosen? Why is it cool? Discuss the general/broader impact of having a good predictive mode. i.e. why is this important? (3 points)

# Figures
Your report should include relevant figures of your choosing to help with the narration of your story, including legends (similar to a scientific paper). For reference you search machine learning and your model in google scholar for reference examples. (3 points)

# Methods Section
This section will include the exploration results, preprocessing steps, models chosen in the order they were executed. You should also describe the parameters chosen. Please make sub-sections for every step. i.e Data Exploration, Preprocessing, Model 1, Model 2, additional models are optional. Please note that models can be the same i.e. DNN but different versions of it if they are distinct enough. Changes can not be incremental. You can put links here to notebooks and/or code blocks using three ` in markup for displaying code. so it would look like this: ``` MY CODE BLOCK ```

*Note: A methods section does not include any why. the reason why will be in the discussion section. This is just a summary of your methods (5 points)*

# Results Section
This will include the results from the methods listed above (C). You will have figures here about your results as well. No exploration of results is done here. This is mainly just a summary of your results. The sub-sections will be the same as the sections in your methods section. (5 points)

# Discussion Section
This is where you will discuss the why, and your interpretation and your though process from beginning to end. This will mimic the sections you have created in your methods section as well as new sections you feel you need to create. You can also discuss how believable your results are at each step. You can discuss any short comings. It's ok to criticize as this shows your intellectual merit, as to how you are thinking about things scientifically and how you are able to correctly scrutinize things and find short comings. In science we never really find the perfect solution, especially since we know something will probably come up int he future (i.e. donkeys) and mess everything up. If you do it's probably a unicorn or the data and model you chose are just perfect for each other! (3 points)

# Conclusion
This is where you do a mind dump on your opinions and possible future directions. Basically what you wish you could have done differently. Here you close with final thoughts. (3 points)

# Statement of Collaboration
This is a statement of contribution by each member. This will be taken into consideration when making the final grade for each member in the group. Did you work as a team? was there a team leader? project manager? coding? writer? etc. Please be truthful about this as this will determine individual grades in participation. There is no job that is better than the other. If you did no code but did the entire write up and gave feedback during the steps and collaborated then you would still get full credit. If you only coded but gave feedback on the write up and other things, then you still get full credit. If you managed everyone and the deadlines and setup meetings and communicated with teaching staff only then you get full credit. **Every role is important as long as you collaborated and were integral to the completion of the project. If the person did nothing, they risk getting a big fat 0.** Just like in any job, if you did nothing, you have the risk of getting fired. Teamwork is one of the most important qualities in industry and academia!

**Format:** Start with Name: Title: Contribution. If the person contributed nothing then just put in writing: Did not participate in the project. (3 points)