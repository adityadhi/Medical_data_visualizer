# Medical_data_visualizer
This project involves visualizing and analyzing medical examination data using Python. The dataset includes information such as age, height, weight, blood pressure, cholesterol levels, glucose levels, smoking habits, alcohol intake, physical activity, and the presence or absence of cardiovascular disease for each patient.

import os
import pandas as pd
import seaborn as sns
import matplotlib.pyplot as plt
import numpy as np
import os

file_path = r'C:\Users\lenovo\OneDrive\Desktop\projects\medical_examination.csv'

print(f"File exists: {os.path.exists(file_path)}")
print(f"File size: {os.path.getsize(file_path)} bytes")

try:
    df = pd.read_csv(file_path)
    print(df.head())  # Print first few rows of data to verify
except pd.errors.EmptyDataError:
    print(f"No columns to parse from file {file_path}")

# File path
file_path = r'C:\Users\lenovo\OneDrive\Desktop\projects\medical_examination.csv'

# Check if file exists and is not empty
if not os.path.exists(file_path):
    raise FileNotFoundError(f"The file {file_path} does not exist.")

if os.path.getsize(file_path) == 0:
    raise ValueError(f"The file {file_path} is empty.")

# Import data
try:
    df = pd.read_csv(file_path)
except pd.errors.EmptyDataError:
    raise ValueError(f"No columns to parse from file {file_path}")

# Add overweight column
df['BMI'] = df['weight'] / ((df['height'] / 100) ** 2)
df['overweight'] = (df['BMI'] > 25).astype(int)

# Normalize cholesterol and gluc values
df['cholesterol'] = df['cholesterol'].apply(lambda x: 0 if x == 1 else 1)
df['gluc'] = df['gluc'].apply(lambda x: 0 if x == 1 else 1)

def draw_cat_plot():
    # Create DataFrame for cat plot using `pd.melt`
    df_cat = pd.melt(df, id_vars=['cardio'], value_vars=['cholesterol', 'gluc', 'smoke', 'alco', 'active', 'overweight'])

    # Group and reformat the data
    df_cat = df_cat.groupby(['cardio', 'variable', 'value']).size().reset_index(name='total')

    # Draw the catplot
    cat_plot = sns.catplot(x='variable', y='total', hue='value', col='cardio', data=df_cat, kind='bar')

    fig = cat_plot.fig
    return fig

def clean_data(df):
    # Clean the data
    df = df[(df['ap_lo'] <= df['ap_hi'])]
    df = df[(df['height'] >= df['height'].quantile(0.025)) & (df['height'] <= df['height'].quantile(0.975))]
    df = df[(df['weight'] >= df['weight'].quantile(0.025)) & (df['weight'] <= df['weight'].quantile(0.975))]
    return df

df_heat = clean_data(df)

def draw_heat_map():
    # Calculate the correlation matrix
    corr = df_heat.corr()

    # Generate a mask for the upper triangle
    mask = np.triu(np.ones_like(corr, dtype=bool))

    # Set up the matplotlib figure
    fig, ax = plt.subplots(figsize=(11, 9))

    # Draw the heatmap
    sns.heatmap(corr, mask=mask, annot=True, fmt='.1f', ax=ax, cmap='coolwarm', center=0, linewidths=.5)

    return fig
