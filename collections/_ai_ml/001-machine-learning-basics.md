---
layout: post
title: "Machine Learning Basics: Supervised Learning and Model Evaluation"
author: "Liberom"
date: 2025-01-08
category: "AI/ML"
toc: true
excerpt: "Master machine learning fundamentals: supervised learning, model evaluation, feature engineering, and common algorithms."
---

# Machine Learning Basics: Supervised Learning and Model Evaluation

Machine learning enables systems to learn from data without explicit programming. Understanding core concepts is essential for building intelligent systems.

## Machine Learning Paradigms

### Supervised Learning

**Definition**: Learning from labeled data (features + correct answers).

**Use Cases**:
- Classification: Predicting categories (spam/not spam, cat/dog)
- Regression: Predicting continuous values (house price, temperature)

```python
from sklearn.datasets import load_iris
from sklearn.model_selection import train_test_split
from sklearn.ensemble import RandomForestClassifier

# Load data
iris = load_iris()
X = iris.data  # Features
y = iris.target  # Labels

# Split into train/test
X_train, X_test, y_train, y_test = train_test_split(
    X, y, test_size=0.2, random_state=42
)

# Train model
model = RandomForestClassifier(n_estimators=100)
model.fit(X_train, y_train)

# Evaluate
score = model.score(X_test, y_test)  # Accuracy
print(f'Accuracy: {score:.2%}')
```

### Unsupervised Learning

**Definition**: Learning patterns from unlabeled data.

**Use Cases**:
- Clustering: Grouping similar items (customer segmentation)
- Dimensionality reduction: Reducing features (principal component analysis)

```python
from sklearn.cluster import KMeans
from sklearn.decomposition import PCA
import numpy as np

# Sample data
X = np.random.randn(100, 20)

# Clustering
kmeans = KMeans(n_clusters=3, random_state=42)
labels = kmeans.fit_predict(X)

# Dimensionality reduction
pca = PCA(n_components=2)
X_reduced = pca.fit_transform(X)
print(f'Explained variance: {pca.explained_variance_ratio_.sum():.2%}')
```

### Reinforcement Learning

**Definition**: Learning through interaction with environment (reward/penalty).

**Use Cases**:
- Game playing (AlphaGo)
- Robotics control
- Resource optimization

## Key Concepts

### Training vs Testing

```python
from sklearn.model_selection import train_test_split

# Never train and test on the same data!
X_train, X_test, y_train, y_test = train_test_split(
    X, y,
    test_size=0.2,          # 80/20 split
    random_state=42,        # Reproducibility
    stratify=y              # Maintain class distribution
)

# Cross-validation for more robust evaluation
from sklearn.model_selection import cross_val_score

scores = cross_val_score(
    model, X_train, y_train,
    cv=5  # 5-fold cross-validation
)
print(f'Mean accuracy: {scores.mean():.2%} (+/- {scores.std():.2%})')
```

### Overfitting and Underfitting

```python
# Overfitting: Model memorizes training data, fails on new data
model_complex = DecisionTreeClassifier(max_depth=20)  # Too deep
model_complex.fit(X_train, y_train)
print(f'Train: {model_complex.score(X_train, y_train):.2%}')  # 99%
print(f'Test: {model_complex.score(X_test, y_test):.2%}')    # 70% (overfitted)

# Underfitting: Model too simple, fails on all data
model_simple = DecisionTreeClassifier(max_depth=2)  # Too shallow
model_simple.fit(X_train, y_train)
print(f'Train: {model_simple.score(X_train, y_train):.2%}')  # 65%
print(f'Test: {model_simple.score(X_test, y_test):.2%}')    # 63% (underfitted)

# Goldilocks: Model with appropriate complexity
model_good = DecisionTreeClassifier(max_depth=5)
model_good.fit(X_train, y_train)
print(f'Train: {model_good.score(X_train, y_train):.2%}')  # 85%
print(f'Test: {model_good.score(X_test, y_test):.2%}')    # 82% (balanced)
```

### Bias-Variance Tradeoff

```
High Bias (Underfitting):
- Simple model
- Fails to capture patterns
- High training AND test error

High Variance (Overfitting):
- Complex model
- Captures noise in training data
- Low training error, high test error

Sweet Spot:
- Balanced model
- Low training error
- Low test error
- Generalizes well
```

## Model Evaluation Metrics

### Classification Metrics

```python
from sklearn.metrics import (
    accuracy_score, precision_score, recall_score, f1_score,
    confusion_matrix, roc_auc_score, roc_curve
)

# Predictions
y_pred = model.predict(X_test)
y_pred_proba = model.predict_proba(X_test)[:, 1]

# Accuracy: Overall correctness
accuracy = accuracy_score(y_test, y_pred)

# Precision: Of predicted positives, how many are correct?
# Use when false positives are costly (spam detection)
precision = precision_score(y_test, y_pred)

# Recall: Of actual positives, how many did we find?
# Use when false negatives are costly (disease detection)
recall = recall_score(y_test, y_pred)

# F1 Score: Harmonic mean of precision and recall
f1 = f1_score(y_test, y_pred)

# Confusion matrix
cm = confusion_matrix(y_test, y_pred)
# [[TN  FP]
#  [FN  TP]]

# ROC-AUC: Handles imbalanced data well
auc = roc_auc_score(y_test, y_pred_proba)

# ROC Curve
fpr, tpr, thresholds = roc_curve(y_test, y_pred_proba)

print(f'Accuracy: {accuracy:.2%}')
print(f'Precision: {precision:.2%}')
print(f'Recall: {recall:.2%}')
print(f'F1: {f1:.2%}')
print(f'ROC-AUC: {auc:.2%}')
```

### Regression Metrics

```python
from sklearn.metrics import mean_squared_error, mean_absolute_error, r2_score
import numpy as np

# Predictions for continuous values
y_pred = model.predict(X_test)

# Mean Absolute Error (MAE): Average absolute difference
mae = mean_absolute_error(y_test, y_pred)

# Mean Squared Error (MSE): Penalizes larger errors more
mse = mean_squared_error(y_test, y_pred)

# Root Mean Squared Error (RMSE): Same units as target
rmse = np.sqrt(mse)

# R² Score: Proportion of variance explained (0-1, higher is better)
r2 = r2_score(y_test, y_pred)

print(f'MAE: {mae:.4f}')
print(f'RMSE: {rmse:.4f}')
print(f'R² Score: {r2:.4f}')
```

## Feature Engineering

### Feature Scaling

```python
from sklearn.preprocessing import StandardScaler, MinMaxScaler

# StandardScaler: Zero mean, unit variance (Gaussian)
scaler = StandardScaler()
X_scaled = scaler.fit_transform(X_train)
X_test_scaled = scaler.transform(X_test)

# MinMaxScaler: Scale to [0, 1]
min_max_scaler = MinMaxScaler()
X_normalized = min_max_scaler.fit_transform(X_train)

# Important: Fit on training data, apply to test data
# Never fit scaler on entire dataset before splitting!
```

### Feature Selection

```python
from sklearn.feature_selection import SelectKBest, f_classif

# Select top K features
selector = SelectKBest(f_classif, k=5)
X_train_selected = selector.fit_transform(X_train, y_train)
X_test_selected = selector.transform(X_test)

# Feature importance from tree-based models
importances = model.feature_importances_
indices = np.argsort(importances)[::-1]

for i in range(5):
    print(f'{i+1}. Feature {indices[i]}: {importances[indices[i]]:.4f}')

# Remove low-importance features
important_features = indices[:10]
X_train_important = X_train[:, important_features]
X_test_important = X_test[:, important_features]
```

### Handling Missing Data

```python
import pandas as pd
from sklearn.impute import SimpleImputer

# Detection
print(df.isnull().sum())

# Strategies
# 1. Drop rows with missing values (lose data)
df.dropna()

# 2. Drop columns with too many missing values
df.dropna(thresh=len(df)*0.8)

# 3. Imputation: Fill with mean/median/forward-fill
imputer = SimpleImputer(strategy='mean')
X_imputed = imputer.fit_transform(X_train)
X_test_imputed = imputer.transform(X_test)

# For categorical data
imputer_cat = SimpleImputer(strategy='most_frequent')
```

## Common Algorithms

### Decision Trees

```python
from sklearn.tree import DecisionTreeClassifier
from sklearn import tree

model = DecisionTreeClassifier(
    max_depth=5,           # Prevent overfitting
    min_samples_split=10,  # Minimum samples to split
    random_state=42
)
model.fit(X_train, y_train)

# Visualize tree
tree.plot_tree(model, feature_names=iris.feature_names)
```

### Random Forest

```python
from sklearn.ensemble import RandomForestClassifier

model = RandomForestClassifier(
    n_estimators=100,      # Number of trees
    max_depth=10,
    random_state=42,
    n_jobs=-1              # Use all cores
)
model.fit(X_train, y_train)

# Feature importance
importances = model.feature_importances_
```

### Logistic Regression

```python
from sklearn.linear_model import LogisticRegression

model = LogisticRegression(
    random_state=42,
    max_iter=1000,
    C=1.0              # Inverse regularization strength
)
model.fit(X_train, y_train)

# Coefficients show feature importance
coefficients = model.coef_[0]
```

### Support Vector Machines (SVM)

```python
from sklearn.svm import SVC

model = SVC(
    kernel='rbf',      # 'linear', 'rbf', 'poly'
    C=1.0,             # Regularization
    gamma='scale',
    random_state=42
)
model.fit(X_train, y_train)
```

## Hyperparameter Tuning

### Grid Search

```python
from sklearn.model_selection import GridSearchCV

# Define parameter grid
param_grid = {
    'max_depth': [3, 5, 7, 10],
    'min_samples_split': [5, 10, 20],
    'n_estimators': [50, 100, 200]
}

# Grid search
grid_search = GridSearchCV(
    RandomForestClassifier(random_state=42),
    param_grid,
    cv=5,
    scoring='f1'
)
grid_search.fit(X_train, y_train)

# Best parameters
print(f'Best parameters: {grid_search.best_params_}')
print(f'Best score: {grid_search.best_score_:.2%}')

# Use best model
best_model = grid_search.best_estimator_
```

### Random Search

```python
from sklearn.model_selection import RandomizedSearchCV

random_search = RandomizedSearchCV(
    RandomForestClassifier(random_state=42),
    param_grid,
    n_iter=10,  # Try 10 random combinations
    cv=5,
    random_state=42
)
random_search.fit(X_train, y_train)
```

## Common Mistakes

```python
# BAD - Data leakage: Scaling entire dataset before split
scaler = StandardScaler()
X_scaled = scaler.fit_transform(X)  # WRONG!
X_train, X_test, y_train, y_test = train_test_split(X_scaled, y)

# GOOD - Scale after split
X_train, X_test, y_train, y_test = train_test_split(X, y)
scaler = StandardScaler()
X_train = scaler.fit_transform(X_train)
X_test = scaler.transform(X_test)

# BAD - Same metric for all problems
# (Accuracy is bad for imbalanced datasets)

# GOOD - Choose metric based on problem
# Imbalanced binary: Use ROC-AUC or F1
# Imbalanced multiclass: Use macro F1
# Balanced data: Accuracy is fine

# BAD - No baseline
model = SomeComplexModel()
print(f'Accuracy: 87%')  # Is this good?

# GOOD - Compare to baseline
baseline_accuracy = (y_test == y_test.mode()).mean()  # Most common class
print(f'Baseline: {baseline_accuracy:.2%}')
print(f'Model: {model.score(X_test, y_test):.2%}')
```

## Conclusion

Machine learning principles:
- Always split data before training
- Evaluate on unseen test data
- Choose appropriate metrics for your problem
- Engineer features carefully
- Use cross-validation for robust evaluation
- Guard against overfitting with regularization
- Establish baselines for comparison
- Monitor for data leakage

