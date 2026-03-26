## Q1. What are types of machine learning?

### Supervised Learning

Uses labeled data, where input and correct output are already known. The model learns to predict outputs for new inputs.

### Example: 
Predicting house prices or classifying emails as spam or not spam.

### Unsupervised Learning

Uses unlabeled data, where the model finds hidden patterns or groups without predefined outputs.

### Example:

Customer segmentation based on purchasing behavior.

### Reinforcement Learning

The model learns by interacting with an environment and receiving rewards or penalties for actions.

### Example:
A robot learning to walk or a game playing agent improving through trials.

### Semi Supervised Learning

Combines a small amount of labeled data with a large amount of unlabeled data.

### Example: 

Image classification when only some images are labeled.

### Self Supervised Learning

The model creates labels from the data itself to learn representations.

### Example: 

Language models predicting missing words in sentences.


## Q2 Coding

```python

regression_data = {
    'Hours_Studied': [1, 2, 3, 4, 5],
    'Sleep_Hours': [7, 6, 5, 4, 3],
    'Score': [50, 55, 65, 70, 80]  # Target variable
}

df = pd.DataFrame(regression_data)


X_reg = df[['Hours_Studied', 'Sleep_Hours']]. # Input

y_reg = df['Score'] # Output
```

## Q3  Difference between regression and classification?


### Regression: 
- Predicts a continuous numeric target, such as house prices, temperature, or salary. 

- The target can take any numeric value within a range, so the model must estimate a number rather than a category.

### Classification: 
- Predicts a categorical target, such as spam/not spam, disease/no disease, or product category.
 
- The target consists of discrete classes, so the model assigns each input to one of the predefined categories.