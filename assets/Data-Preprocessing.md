### [Back to readme](../README.md)

---

# Data Preprocessing Notebook

This repository contains the `Preprocessing.ipynb` notebook, which is designed for preprocessing data before feeding it into a machine learning model. The preprocessing steps include data loading, cleaning, transformation, and preparation for training.

## Table of Contents
1. [Installation](#installation)
2. [Usage](#usage)
3. [Notebook Overview](#notebook-overview)
    - [Step 1: Import Libraries](#step-1-import-libraries)
    - [Step 2: Load Data](#step-2-load-data)
    - [Step 3: Data Cleaning](#step-3-data-cleaning)
    - [Step 4: Data Transformation](#step-4-data-transformation)
    - [Step 5: Data Preparation](#step-5-data-preparation)
    - [Step 6: Data Tokenization](#step-6-data-tokenization)
    - [Step 7: DataLoader Preparation](#step-7-dataloader-preparation)

## Installation
To run this notebook, you'll need Python and several Python libraries. Follow these steps to set up your environment:

1. Clone the repository:
    ```sh
    git clone https://github.com/DiodDan/Big-data-Analytics.git
    cd Big-data-Analytics
    ```

2. Create a virtual environment:
    ```sh
    python3 -m venv venv
    source venv/bin/activate   # On Windows use `venv\Scripts\activate`
    ```

3. Install the required libraries:
    ```sh
    pip install pandas numpy nltk transformers torch scikit-learn
    ```

## Usage

To use the notebook, follow these steps:

1. Activate your virtual environment (if not already activated):
    ```sh
    source venv/bin/activate   # On Windows use `venv\Scripts\activate`
    ```

2. Launch Jupyter Notebook:
    ```sh
    jupyter notebook
    ```

3. Open `Preprocessing.ipynb` and run the cells sequentially.

## Notebook Overview

### Step 1: Import Libraries

In the first step, all necessary libraries are imported. These include libraries for data manipulation (`pandas`, `numpy`), text processing (`nltk`), and handling the transformer model (`transformers`, `torch`).
```python
import pandas as pd
import numpy as np
import nltk
from transformers import BertTokenizer
import torch
from torch.utils.data import DataLoader, RandomSampler, SequentialSampler, TensorDataset
```

### Step 2: Load Data

Data is loaded into a pandas DataFrame. This step includes reading data from CSV files and displaying the first few rows to understand the structure.

```python
nvidia_frame = pd.read_csv("nvidia_data.csv")
print(nvidia_frame.head())
```

### Step 3: Data Cleaning

Data cleaning is an essential step in preprocessing. This step involves Removing Duplicates and Text Cleaning.

**Removing Duplicates**
```python
nvidia_frame.drop_duplicates(subset='content', inplace=True) 
competitors_frame.drop_duplicates(subset='content', inplace=True)
```
**Text Cleaning**
```python
del nvidia_frame['#']
del competitors_frame['#']
```
```python
def strip_emoji(text):
    RE_EMOJI = re.compile('[\U00010000-\U0010ffff]', flags=re.UNICODE)
    return RE_EMOJI.sub(r'', text)

def strip_all_entities(text): 
    text = text.replace('\r', '').replace('\n', ' ').replace('\n', ' ').lower()
    text = re.sub(r"(?:\@|https?\://)\S+", "", text) 
    text = re.sub(r'[^\x00-\x7f]',r'', text) 
    banned_list= string.punctuation + 'Ã'+'±'+'ã'+'¼'+'â'+'»'+'§'
    table = str.maketrans('', '', banned_list)
    text = text.translate(table)
    return text

def clean_hashtags(tweet):
    new_tweet = " ".join(word.strip() for word in re.split('#(?!(?:hashtag)\b)[\w-]+(?=(?:\s+#[\w-]+)*\s*$)', tweet)) 
    new_tweet2 = " ".join(word.strip() for word in re.split('#|_', new_tweet))
    return new_tweet2

def filter_chars(a):
    sent = []
    for word in a.split(' '):
        if ('$' in word) | ('&' in word):
            sent.append('')
        else:
            sent.append(word)
    return ' '.join(sent)

def remove_mult_spaces(text):
    return re.sub("\s\s+" , " ", text)
```

### Step 4: Data Transformation

The 'created_at' column is converted to datetime, and the difference in days from the latest date in the dataset is calculated.

```python
nvidia_frame['created_at'] = pd.to_datetime(nvidia_frame['created_at'])
max_date = nvidia_frame['created_at'].max()
nvidia_frame['days_since'] = (max_date - nvidia_frame['created_at']).dt.days
```

### Step 5: Data Preparation

The dataset is split into training and validation sets.

```python
from sklearn.model_selection import train_test_split

train_data, val_data = train_test_split(nvidia_frame, test_size=0.2, random_state=42)
```

### Step 6: Data Tokenization

The `BertTokenizer` is used to tokenize the text data. This step converts text into tokens that are suitable for input into the BERT model.

```python
tokenizer = BertTokenizer.from_pretrained('bert-base-uncased', do_lower_case=True)

def encode_data(data):
    input_ids = []
    attention_masks = []
    for text in data:
        encoded = tokenizer.encode_plus(
            text,
            add_special_tokens=True,
            max_length=64,
            padding='max_length',
            truncation=True,
            return_attention_mask=True,
            return_tensors='pt'
        )
        input_ids.append(encoded['input_ids'])
        attention_masks.append(encoded['attention_mask'])
    
    return torch.cat(input_ids, dim=0), torch.cat(attention_masks, dim=0)

train_inputs, train_masks = encode_data(train_data['post_text'].values)
val_inputs, val_masks = encode_data(val_data['post_text'].values)
```

### Step 7: DataLoader Preparation

The tokenized data is prepared for the model using DataLoader, which will handle batching and shuffling.

```python
train_labels = torch.tensor(train_data['label'].values)
val_labels = torch.tensor(val_data['label'].values)

batch_size = 4

train_data = TensorDataset(train_inputs, train_masks, train_labels)
train_sampler = RandomSampler(train_data)
train_dataloader = DataLoader(train_data, sampler=train_sampler, batch_size=batch_size)

val_data = TensorDataset(val_inputs, val_masks, val_labels)
val_sampler = SequentialSampler(val_data)
val_dataloader = DataLoader(val_data, sampler=val_sampler, batch_size=batch_size)
```
[Back to top](#data-preprocessing-notebook)
