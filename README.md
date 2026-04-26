# Network Anomaly Detection using XGBoost

## Project Overview
This project enhances the performance of an intrusion detection system using the **XGBoost classifier** through systematic hyperparameter tuning. The project leverages the **CSE-CIC-IDS 2018 dataset**, a benchmark dataset in the cybersecurity domain for network intrusion detection and classification. The implementation employs **GridSearchCV** and **RandomizedSearchCV** to optimize hyperparameters and improve model performance metrics including accuracy, precision, recall, and F1-score.

## Dataset Description
The project uses the **CSE-CIC-IDS 2018 dataset** from Kaggle. This dataset contains network traffic logs labeled as benign or malicious across multiple attack categories.

**Dataset Link:** [Kaggle CSE-CIC-IDS 2018](https://www.kaggle.com/datasets/solarmainframe/ids-intrusion-csv)

**Key Data Characteristics:**
- Downloaded directly from Kaggle using `kagglehub`
- Contains network flow statistics and attack labels
- Multiple attack types including FTP-BruteForce, SSH-Bruteforce, and benign traffic
- Raw dataset dimensions: 1,048,575 rows, initially 80 features
- Final dataset after preprocessing: 36 features + 1 target variable
- Train-test split: 70% training, 30% testing with stratification

## Project Objectives
1. **Build and optimize** an XGBoost classifier for network intrusion detection
2. **Systematically tune hyperparameters** using GridSearchCV and RandomizedSearchCV
3. **Compare performance** of three modeling approaches: Baseline, GridSearch, and RandomSearch
4. **Evaluate model quality** using accuracy, precision, recall, F1-score, classification reports, and confusion matrices
5. **Analyze feature importance** to identify the most predictive network characteristics
6. **Handle class imbalance** between benign and attack traffic using balanced class weights

## Pipeline Overview

### 1. Data Loading
- Downloads the **IDS Intrusion CSV dataset from Kaggle** (specifically the Feb 14, 2018 capture file)
- Contains network flow records labeled as benign or malicious (various attack types)
- Uses `kagglehub` library for automated dataset retrieval
- Initial dataset dimensions: 1,048,575 rows × 80 features

### 2. Data Cleaning

The data cleaning pipeline removes irrelevant and problematic features:

- **Detects and handles missing values** — identifies columns with null/NaN values
- **Drops the Flow Byts/s column** — contains invalid values (inf, Infinity) that cannot be used
- **Removes zero-variance (constant) columns** — features that carry no information (all same value)
- **Removes duplicate columns** — identifies and drops redundant columns with identical values
  - Example duplicates: SYN Flag Cnt, Fwd Seg Size Avg, Subflow Fwd Pkts, Subflow Fwd Byts, Subflow Bwd Pkts, Subflow Bwd Byts
- **Drops the Timestamp column** — non-numeric temporal feature, not useful for classification
- **Removes duplicate rows** — eliminates exact duplicate records

**Result after cleaning:** 62 features (reduced from 80)

### 3. Feature Engineering

Prepares features and target variable for modeling:

- **Merges attack labels:** Combines 'FTP-BruteForce' and 'SSH-Bruteforce' into single 'BruteForce' class
  - Simplifies to binary classification problem: Benign vs. Malicious (BruteForce/Attacks)
- **Label encodes the target column (Label):** Converts categorical labels to numeric values (0, 1, 2, ...)
  - Uses `sklearn.preprocessing.LabelEncoder` for encoding
- **Plots Pearson correlation heatmap** of top 20 features
  - Identifies highly correlated feature pairs for potential removal
- **Drops highly correlated features** (threshold: 0.90)
  - Reduces multicollinearity and redundant information
  - Features above 0.90 correlation are removed
- **Removes duplicate rows** — eliminates any remaining duplicate records

**Result:** Cleaned dataset with 36 independent features and 1 encoded target variable

### 4. Class Imbalance Handling

Addresses the inherent imbalance in network traffic (majority benign, minority attacks):

- **Computes class weights** using `sklearn.utils.class_weight.compute_class_weight()`
  - Automatically calculates balanced weights based on class frequency
- **Applies scale_pos_weight in XGBoost** — adjusts loss function to penalize minority class misclassification more heavily
- **Result:** Model gives appropriate emphasis to minority attack class during training, preventing bias toward benign traffic

### 5. Model Training — Three Approaches

The project implements and compares three distinct training strategies:

| Approach | Description | Hyperparameters Tested | Cross-Validation | Computation Time |
|----------|-------------|----------------------|------------------|-----------------|
| **Baseline XGBoost** | Default model with scale_pos_weight for imbalance handling | None (default params) | None (single train) | ~5-10 minutes |
| **GridSearchCV** | Exhaustive hyperparameter tuning over predefined grid | n_estimators: [100,200,300]<br>learning_rate: [0.01,0.1,0.2]<br>max_depth: [3,5,7]<br>subsample: [0.7,0.8,0.9]<br>colsample_bytree: [0.7,0.8,0.9]<br>**Total: 243 combinations** | 3-fold stratified | 30-60 minutes |
| **RandomizedSearchCV** | Faster random sampling of hyperparameter space | Same distribution as GridSearchCV | 3-fold stratified (10 iterations) | 10-30 minutes |

**Configuration Details:**
- **Objective:** `binary:logistic` (binary classification)
- **Evaluation Metric:** `logloss`
- **Random State:** 12 (ensures reproducibility)
- **Test Split:** 30% test, 70% training (stratified to maintain class distribution)
- **Parallel Processing:** n_jobs=-1 (uses all available CPU cores)

### 6. Evaluation

Each model is evaluated using comprehensive metrics:

- **Accuracy Score** — overall correctness of predictions on test set
- **Classification Report** — per-class metrics including:
  - Precision: True positives / (True positives + False positives)
  - Recall: True positives / (True positives + False negatives)
  - F1-Score: Harmonic mean of precision and recall
  - Support: Number of samples per class
- **Confusion Matrix** — breakdown of prediction types:
  - True Positives (TP): Correctly identified attacks
  - True Negatives (TN): Correctly identified benign traffic
  - False Positives (FP): Benign traffic falsely flagged as attack (false alarms)
  - False Negatives (FN): Attacks missed by model (critical failures)

### 7. Feature Importance

Analyzes which network flow characteristics are most predictive:

- **Extracts feature importances** from the best-trained model (from RandomizedSearchCV)
- **Ranks features by importance score** in descending order
- **Visualizes top 5-10 most important features** using a bar chart
- **Insights:** Shows which network flow characteristics (e.g., bytes transferred, packet counts, protocol flags) best distinguish benign from malicious traffic

---

## Dependencies
```
pandas          # Data manipulation and analysis
numpy           # Numerical computing
scikit-learn    # Machine learning library (GridSearchCV, metrics, preprocessing)
xgboost         # Gradient boosting classifier
seaborn         # Statistical data visualization
matplotlib      # Plotting and visualization
kagglehub       # Kaggle dataset download
```

## Installation & Setup

### Prerequisites
- Python 3.7+
- pip or conda package manager
- Kaggle account (for dataset access via kagglehub)
- ~4GB RAM minimum (for dataset processing)

### Steps to Run
1. Clone the repository:
   ```bash
   git clone https://github.com/tushargupta1111/Network-Intrusion-Detection.git
   cd Network-Intrusion-Detection
   ```

2. Install required packages:
   ```bash
   pip install pandas numpy scikit-learn xgboost seaborn matplotlib kagglehub
   ```

3. Open and run the Jupyter notebook:
   ```bash
   jupyter notebook "XGBoosting Classifier.ipynb"
   ```

4. Execute cells sequentially from top to bottom (Shift+Enter to run each cell)

## Key Implementation Details

### Data Preprocessing Pipeline Flow
```
Raw Data (80 features)
    ↓
Handle Missing Values
    ↓
Remove Invalid Columns (Flow Byts/s)
    ↓
Remove Zero-Variance Features
    ↓
Remove Duplicate Columns
    ↓
Drop Timestamp Column
    ↓
Remove Duplicate Rows
    ↓
Label Encode Target Variable
    ↓
Feature Correlation Analysis
    ↓
Remove Highly Correlated Features (threshold: 0.90)
    ↓
Final Dataset (36 features + 1 target)
```

### Model Comparison Strategy
1. **Baseline XGBoost** provides performance baseline with default parameters
2. **GridSearchCV** exhaustively searches 243 parameter combinations for optimal configuration
3. **RandomizedSearchCV** efficiently samples 10 combinations, balancing speed and quality

### Class Imbalance Handling
- Network traffic typically has huge class imbalance (e.g., 99% benign, 1% attacks)
- Calculated class weights adjust XGBoost's loss function to treat minority class (attacks) more heavily
- Results in better attack detection (higher recall) while minimizing false alarms (maintaining precision)

## Model Performance Metrics

### Test Set Evaluation
All three models evaluated on same 30% test set with:
- **Accuracy:** Percentage of correct predictions
- **Precision:** Of predicted attacks, how many were correct (minimize false alarms)
- **Recall:** Of actual attacks, how many were detected (minimize missed attacks)
- **F1-Score:** Balanced metric combining precision and recall
- **Confusion Matrix:** Visual breakdown of TP, TN, FP, FN for detailed analysis

### Expected Results
- Baseline likely has moderate performance
- GridSearchCV typically achieves best performance through exhaustive search
- RandomizedSearchCV balances computational cost with near-optimal performance

## File Structure
```
Network-Intrusion-Detection/
├── README.md                          # This file - project documentation
├── XGBoosting Classifier.ipynb       # Main Jupyter notebook with complete pipeline implementation
```

## Notebook Cell Organization
The notebook implements the pipeline in sequential cells:
1. **Import Libraries** - Pandas, NumPy, scikit-learn, XGBoost, visualization libraries
2. **Data Loading** - Download and load IDS dataset from Kaggle
3. **Data Cleaning** - Execute all cleaning steps (missing values, invalid columns, duplicates)
4. **Feature Engineering** - Label encoding, correlation analysis, feature selection
5. **Class Imbalance Handling** - Compute and display balanced class weights
6. **Train-Test Split** - Stratified 70-30 split with random_state=12
7. **Baseline XGBoost** - Train default model and evaluate
8. **GridSearchCV** - Exhaustive hyperparameter tuning with 3-fold CV
9. **RandomizedSearchCV** - Random sampling tuning with 3-fold CV
10. **Feature Importance** - Visualize top features from best model

## How to Extend This Project
1. **Try more hyperparameter ranges** for deeper GridSearchCV or more RandomizedSearchCV iterations
2. **Experiment with preprocessing techniques** (StandardScaler, RobustScaler, PCA)
3. **Implement ensemble methods** (combine multiple tuned models)
4. **Add explainability analysis** (SHAP values, LIME for local interpretability)
5. **Deploy model** (save with pickle/joblib, create REST API)
6. **Compare with other models** (Random Forest, LightGBM, CatBoost, Neural Networks)
7. **Add threshold optimization** for tuning precision vs. recall trade-off
8. **Generate ROC curves and AUC metrics** for comprehensive evaluation

## Future Improvements
- Deep learning approaches (LSTM, autoencoders for anomaly detection)
- Real-time streaming detection pipeline
- Model explainability (SHAP, attention mechanisms)
- Evaluation on additional datasets (NSL-KDD, UNSW-NB15, CICIDS2019)
- Production deployment with monitoring and retraining triggers
- Automated hyperparameter optimization (Optuna, Ray Tune)

## Key Insights from Notebook Analysis
1. **Data Quality:** 43 features removed (54% of original 80) due to invalidity or redundancy
2. **Feature Redundancy:** Pearson correlation revealed several highly correlated feature pairs
3. **Class Imbalance:** Network traffic heavily skewed toward benign traffic
4. **Hyperparameter Impact:** Different parameter combinations significantly affect accuracy
5. **Top Features:** A small set of network flow characteristics are highly predictive

## References
- [XGBoost Documentation](https://xgboost.readthedocs.io/)
- [Scikit-learn Model Selection](https://scikit-learn.org/stable/modules/model_selection.html)
- [CSE-CIC-IDS 2018 Dataset on Kaggle](https://www.kaggle.com/datasets/solarmainframe/ids-intrusion-csv)
- [Handling Imbalanced Data](https://scikit-learn.org/stable/modules/generated/sklearn.utils.class_weight.compute_class_weight.html)

## Author
**Tushar Gupta**  
GitHub: [@tushargupta1111](https://github.com/tushargupta1111)

## License
Open source - available for research and educational purposes.

## Contributing
Contributions welcome! Please submit pull requests or open issues for improvements and bug reports.
