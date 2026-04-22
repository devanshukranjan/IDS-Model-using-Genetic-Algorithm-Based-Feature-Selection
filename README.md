# IDS Model using Genetic Algorithm-Based Feature Selection

This README explains the IDS notebook:

`ids-with-ga-feature-selection-over-bot-iot.ipynb`

It also explains the preprocessing notebook used to create that IDS notebook's input dataset.

The notebook builds an Intrusion Detection System (IDS) for the Bot-IoT dataset. It uses a Genetic Algorithm (GA) to select the most useful network traffic features, then trains an XGBoost multiclass classifier to classify traffic into attack or normal categories.

## Project Flow

The full flow for the dataset used by this IDS notebook is:

```text
Raw Bot-IoT 5% dataset files downloaded from the official source
        |
        v
pre-processing-bot-iot.ipynb
        |
        |-- Load and combine raw CSV files
        |-- Inspect shape, data types, null values, and class labels
        |-- Drop attack and subcategory columns
        |-- Label-encode categorical feature columns
        |-- Standard-scale all feature columns
        |-- Save the preprocessed dataset
        v
Datasets/bot_iot_5_preprocessed_dataset.csv
        |
        v
ids-with-ga-feature-selection-over-bot-iot.ipynb
        |
        |-- Encode target category labels
        |-- Run GA-based feature selection
        |-- Save GA-selected dataset
        |-- Train and evaluate XGBoost IDS model
        v
Datasets/bot_iot_5_ga_feature_selection_dataset.csv
```

## Preprocessing Notebook

The preprocessing steps used to create the dataset for `ids-with-ga-feature-selection-over-bot-iot.ipynb` are present in:

```text
pre-processing-bot-iot.ipynb
```

## Preprocessing Steps

The IDS notebook does not start from the raw Bot-IoT files directly. It starts from a preprocessed CSV. That CSV is created by `pre-processing-bot-iot.ipynb`.

### 1. Load Raw Bot-IoT Files

The preprocessing notebook loads raw Bot-IoT 5% CSV files downloaded from the official source. In this project, those files are read from the local project folder when available:

```python
Path("5%/All features")
```

If the notebook is run on Kaggle instead of this local project folder, it can also use:

```python
Path("/kaggle/input/bot-iot-5-data")
```

It walks through the folder, reads each CSV file, and concatenates all rows into one dataframe:

```python
df = pd.concat([df, temp_df], axis=0, ignore_index=True)
```

After loading, the raw combined dataset has:

```text
Rows: 3,668,522
Columns: 46
```

### 2. Inspect Dataset Structure

The notebook checks:

- Dataset shape.
- First few rows using `df.head()`.
- Column names and data types using `df.info()` and `df.dtypes`.
- Missing values using `df.isnull().sum()`.
- Unique values in `category`, `subcategory`, and `attack`.

The null-value check shows that the columns do not contain missing values, so no missing-value imputation or row deletion is applied.

The label-related columns contain:

```text
category: DDoS, DoS, Normal, Reconnaissance, Theft
subcategory: UDP, HTTP, TCP, Normal, OS_Fingerprint, Service_Scan, Data_Exfiltration, Keylogging
attack: 1, 0
```

### 3. Separate Features and Target

The preprocessing notebook keeps `category` as the final multiclass target label.

It removes these columns from the feature matrix:

```text
category
attack
subcategory
```

This is done because:

- `category` is the target label to be predicted.
- `attack` is a binary label and would duplicate target information.
- `subcategory` is another label-like field and would leak attack-type information into the feature set.

The code is:

```python
X = df.drop(['category', 'attack', 'subcategory'], axis=1)
y = df['category']
```

After this step:

```text
Feature columns: 43
Target column: category
```

### 4. Encode Categorical Feature Columns

Some feature columns are object/string columns, such as protocol, address, port, flag, and state-related fields. Machine learning models need numeric values, so the notebook applies `LabelEncoder` to every object-type feature column.

```python
for col in X.select_dtypes(include=['object']):
    X[col] = le.fit_transform(X[col])
```

This converts values such as IP addresses, protocol names, flags, ports, and state labels into integer codes.

### 5. Scale Features

After encoding categorical columns, the notebook standardizes all feature columns using `StandardScaler`.

```python
scaler = StandardScaler()
X_scaled = pd.DataFrame(scaler.fit_transform(X), columns=X.columns)
```

Standard scaling converts each feature to a distribution with approximately:

```text
Mean: 0
Standard deviation: 1
```

This helps make features comparable for correlation calculation and feature-selection scoring.

### 6. Recombine Features and Target

The scaled features are combined with the original `category` labels:

```python
combined_df = pd.concat([X_processed, y_processed], axis=1)
```

The target labels remain as class names, not numbers, at this preprocessing stage.

### 7. Save the Preprocessed Dataset

The preprocessing notebook saves the final preprocessed dataset as:

```python
combined_df.to_csv("Datasets/bot_iot_5_preprocessed_dataset.csv", index=False)
```

In this project folder, that generated file is available at:

```text
Datasets/bot_iot_5_preprocessed_dataset.csv
```

Download link:

```text
https://1024terabox.com/s/1zpKP-cYJjDANqhfbTQcOGQ
```

This is the exact file loaded by `ids-with-ga-feature-selection-over-bot-iot.ipynb`.

## Project Objective

The project performs multiclass intrusion detection on Bot-IoT traffic. Instead of training the model on every available feature, the notebook first searches for a smaller and stronger feature subset using a Genetic Algorithm.

The main goals are:

- Load the preprocessed Bot-IoT dataset.
- Encode the attack category labels.
- Use GA-based feature selection to reduce the original feature set.
- Save the GA-selected dataset.
- Train an XGBoost classifier using the selected features.
- Evaluate the classifier using accuracy, confusion matrix, and classification report.

## Dataset

The notebook uses this local dataset:

```text
Datasets/bot_iot_5_preprocessed_dataset.csv
```

Dataset download link:

```text
https://1024terabox.com/s/1zpKP-cYJjDANqhfbTQcOGQ
```

The input dataset has:

```text
Rows: 3,668,522
Columns: 44
```

There are 43 input feature columns and 1 target column named `category`.

The target classes are:

| Class | Meaning in this project | Records |
| --- | --- | ---: |
| DDoS | Distributed Denial of Service attack traffic | 1,926,624 |
| DoS | Denial of Service attack traffic | 1,650,260 |
| Reconnaissance | Scanning/probing/reconnaissance traffic | 91,082 |
| Normal | Normal network traffic | 477 |
| Theft | Data theft related traffic | 79 |

The dataset is highly imbalanced. Most records belong to DDoS and DoS, while Normal and Theft have very few records.

The original feature columns include packet identifiers, time fields, protocol fields, source/destination address and port fields, packet/byte counts, duration statistics, rate statistics, and connection-count features.

## Output Dataset

After GA feature selection, the notebook saves a reduced dataset:

```text
Datasets/bot_iot_5_ga_feature_selection_dataset.csv
```

The output dataset has:

```text
Rows: 3,668,522
Columns: 23
```

It contains 22 selected features plus the target `category` column.

Selected features:

```text
pkSeqID
stime
proto_number
saddr
sport
daddr
dport
pkts
ltime
stddev
min
max
sbytes
rate
drate
TnBPDstIP
AR_P_Proto_P_SrcIP
AR_P_Proto_P_DstIP
N_IN_Conn_P_DstIP
N_IN_Conn_P_SrcIP
AR_P_Proto_P_Sport
AR_P_Proto_P_Dport
```

## Overall Architecture

The notebook follows this architecture:

```text
Preprocessed Bot-IoT CSV
        |
        v
Load dataset with pandas
        |
        v
Encode category labels with LabelEncoder
        |
        v
Separate features X and target y
        |
        v
Genetic Algorithm feature selection
        |
        |-- Create binary feature chromosomes
        |-- Evaluate each chromosome with correlation merit and XGBoost G-mean
        |-- Select, crossover, and mutate chromosomes
        |-- Keep best feature subsets over generations
        v
Best selected feature subset
        |
        v
Save reduced GA-selected dataset
        |
        v
Train final XGBoost classifier
        |
        v
Evaluate with accuracy, confusion matrix, and classification report
```

## Notebook Workflow

### 1. Import Libraries

The notebook imports:

- `pandas` and `numpy` for data handling.
- `random` for GA parent selection, crossover, and mutation.
- `scikit-learn` for label encoding, train/test split, confusion matrix, classification report, recall-related metrics, and accuracy.
- `xgboost.XGBClassifier` for machine learning classification.
- `scipy.stats.pearsonr` for correlation-based feature merit calculation.
- `matplotlib` and `seaborn` for visualizations.

### 2. Load and Prepare the Dataset

The dataset is loaded from:

```python
df = pd.read_csv('Datasets/bot_iot_5_preprocessed_dataset.csv')
```

The target column is `category`.

The notebook uses `LabelEncoder` to convert category names into numeric labels because XGBoost requires numeric target values.

```python
df["category_encoded"] = label_encoder.fit_transform(df["category"])
y = df["category_encoded"]
X = df.drop(columns=["category", "category_encoded"])
```

The model learns from `X`, while `y` contains encoded class labels.

## Genetic Algorithm for Feature Selection

The GA is used to search for the best subset of features. Each possible feature subset is represented as a chromosome.

### Chromosome Representation

Each chromosome is a binary list with one bit per feature:

```text
1 = feature is selected
0 = feature is not selected
```

Because the original dataset has 43 input features, each chromosome has 43 binary values.

Example:

```text
[1, 0, 1, 1, 0, ...]
```

This means the GA selects only the features where the bit value is `1`.

### GA Parameters

The notebook uses these GA settings:

| Parameter | Value | Purpose |
| --- | ---: | --- |
| `POPULATION_SIZE` | 50 | Number of chromosomes maintained per generation |
| `MAX_GENERATIONS` | 100 | Maximum number of GA iterations |
| `MAX_NO_IMPROVEMENT` | 10 | Early stopping limit if best fitness does not improve |
| `a` | 0.5 | Weight between correlation merit and model performance |
| Mutation rate | 0.4 | Probability of flipping each chromosome bit |
| Children per generation | 20% of population | New chromosomes generated each generation |

### Initial Population

The initial population is created randomly:

```python
np.random.randint(0, 2, n_features)
```

This creates different random feature subsets at the start of the search.

### Fitness Function

The fitness function decides how good a feature subset is.

For each chromosome:

1. Select the feature columns where the chromosome bit is `1`.
2. Split the selected data into training and testing sets.
3. Train an XGBoost classifier.
4. Predict the test set.
5. Compute sensitivity, specificity, and G-mean from the confusion matrix.
6. Compute correlation-based feature merit.
7. Combine both values into one final fitness score.

The notebook uses this fitness formula:

```text
fitness = 0.5 * correlation_merit + 0.5 * gmean
```

This means the GA does not select features only because they improve model accuracy. It also rewards feature subsets that have strong correlation with the target class and lower redundancy among selected features.

### Correlation-Based Feature Merit

The notebook calculates feature quality using Pearson correlation.

It measures:

- Feature-class correlation: how strongly each selected feature relates to the target class.
- Feature-feature correlation: how strongly selected features relate to each other.

A good feature subset should:

- Have high correlation with the target class.
- Avoid selecting many features that are highly correlated with each other.

The merit calculation used in the notebook is:

```text
merit = (k * mean_feature_class_correlation)
        / sqrt(k + (k - 1) * mean_feature_feature_correlation)
```

Where `k` is the number of selected features.

### Model-Based GA Evaluation

Inside the GA fitness function, the notebook trains this model:

```python
XGBClassifier(use_label_encoder=False, eval_metric='mlogloss')
```

The model is trained on the selected feature subset. Its predictions are used to calculate:

- Sensitivity
- Specificity
- G-mean

G-mean is useful for imbalanced datasets because it considers performance across classes instead of only rewarding the majority classes.

### Caching Fitness Scores

The notebook uses a dictionary named `fitness_cache`.

This avoids recalculating the same chromosome multiple times:

```python
fitness_cache[key] = fitness
```

This is important because training XGBoost repeatedly on millions of rows is computationally expensive.

### Selection

The notebook selects two random parents from the population:

```python
random.sample(parents, 2)
```

These parents are used to create children.

### Crossover

The notebook uses uniform crossover.

For every bit position, the child randomly takes the bit from either parent.

This mixes feature choices from two parent chromosomes and creates new feature subsets.

### Mutation

Mutation randomly flips chromosome bits:

```text
0 becomes 1
1 becomes 0
```

The mutation rate is `0.4`, so each bit has a 40% chance of being flipped.

Mutation helps the GA explore new feature subsets and avoid getting stuck too early.

### Population Update

For every generation:

1. Parent population fitness is calculated.
2. Children are created using selection, crossover, and mutation.
3. Children fitness is calculated.
4. Parents and children are combined.
5. The best 50 chromosomes are kept for the next generation.

This is an elitist replacement strategy because the best-performing chromosomes survive.

### Stopping Condition

The GA stops when either:

- It reaches 100 generations, or
- The best score does not improve for 10 consecutive generations.

In the saved notebook output, the GA ran for 21 generations before stopping.

## Machine Learning Algorithm Used

The ML algorithm used in the notebook is:

```text
XGBoost Classifier
```

XGBoost is a gradient boosting algorithm that builds an ensemble of decision trees. It is useful for tabular datasets because it can model nonlinear relationships and interactions between features.

The notebook uses XGBoost in two places:

1. During GA fitness evaluation, to score each selected feature subset.
2. After GA completes, to train the final IDS model using the best selected features.

Final model setup:

```python
model = XGBClassifier(use_label_encoder=False, eval_metric='mlogloss')
```

The final train/test split is:

```text
Training set: 70%
Testing set: 30%
random_state: 42
```

## Evaluation Results

The notebook reports:

```text
Training Accuracy: 1.0000
Testing Accuracy: 1.0000
```

Classification report on the test set:

| Class | Precision | Recall | F1-score | Support |
| --- | ---: | ---: | ---: | ---: |
| DDoS | 1.00 | 1.00 | 1.00 | 578,162 |
| DoS | 1.00 | 1.00 | 1.00 | 495,064 |
| Normal | 1.00 | 1.00 | 1.00 | 141 |
| Reconnaissance | 1.00 | 1.00 | 1.00 | 27,166 |
| Theft | 1.00 | 1.00 | 1.00 | 24 |

Overall test accuracy:

```text
1.00
```

The notebook also plots:

- Fitness distribution per generation.
- Parent vs child fitness comparison.
- GA convergence plot.
- Final confusion matrix for the selected feature subset.

## What the Project Performs

In simple terms, this project performs the following:

1. Takes a preprocessed Bot-IoT network traffic dataset.
2. Treats `category` as the class label for IDS classification.
3. Uses a Genetic Algorithm to find a smaller feature subset.
4. Scores each feature subset using both correlation statistics and XGBoost classification performance.
5. Saves the best selected features into a new CSV file.
6. Trains an XGBoost IDS classifier using only those selected features.
7. Evaluates the classifier against five traffic categories: DDoS, DoS, Normal, Reconnaissance, and Theft.

## Files Used by This Notebook

| File | Purpose |
| --- | --- |
| `pre-processing-bot-iot.ipynb` | Creates the preprocessed Bot-IoT dataset used as input by the IDS notebook |
| `ids-with-ga-feature-selection-over-bot-iot.ipynb` | Main notebook explained in this README |
| `Datasets/bot_iot_5_preprocessed_dataset.csv` | Input preprocessed Bot-IoT dataset. Download: https://1024terabox.com/s/1zpKP-cYJjDANqhfbTQcOGQ |
| `Datasets/bot_iot_5_ga_feature_selection_dataset.csv` | Output dataset after GA feature selection |

## Important Notes

- The notebook works on an already preprocessed dataset. Encoding and scaling of raw Bot-IoT fields happened before this notebook.
- The final model reports perfect accuracy on the notebook's saved run. Because the dataset is very imbalanced and the score is extremely high, the result should be validated carefully before claiming real-world generalization.
- The GA is computationally expensive because each chromosome evaluation trains an XGBoost model.
- The selected feature set may change in another run because the GA uses random initialization, random parent selection, crossover, and mutation without a fixed global random seed for every random source.


## Important Notes

- The notebook works on an already preprocessed dataset. Encoding and scaling of raw Bot-IoT fields happened before this notebook.
- The final model reports perfect accuracy on the notebook's saved run. Because the dataset is very imbalanced and the score is extremely high, the result should be validated carefully before claiming real-world generalization.
- The GA is computationally expensive because each chromosome evaluation trains an XGBoost model.
- The selected feature set may change in another run because the GA uses random initialization, random parent selection, crossover, and mutation without a fixed global random seed for every random source.
