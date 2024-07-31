<!--
SPDX-FileCopyrightText: 2024 Johannes Unruh <johannes.unruh@dlr.de>

SPDX-License-Identifier: CC-BY-4.0
-->

# Introduction

Welcome to the documentation for the Federated Learning Demonstrator Frontend, developed by the German Aerospace Center (DLR). This tool helps manage and monitor federated learning models across distributed environments, providing insights into training processes and outcomes across various nodes.

## Features

### Dashboard

The dashboard offers a comprehensive view of federated learning activities:

- **Model Metrics Visualization**: Real-time visualization of model metrics.
- **Participants**: Display users participanting in the training.
- **Training Contribution**: Tracks each participants input, showing their impact on training.

## Usage

### Navigating the Dashboard

The dashboard serves as the control center for managing federated learning activities, including:

- **Interpreting Data Visualizations**: Understand various graphs and charts to track model performance.

### Creating a Model

To create a new federated learning model:

1. **Go to the Admin Section**: Select 'Create Model'.
2. **Input Model Parameters**: Provide a name, description, and upload the model in [TorchScript](https://pytorch.org/tutorials/beginner/Intro_to_TorchScript_tutorial.html) format.
3. **Finalize**: Click 'Create' to set up the model.

### Initiating a Training Process

To start training:

1. **Admin Section**: Choose 'Create Training'.
2. **Training Configuration**: Select the model, set parameters, and designate participating nodes.
3. **Launch**: Click 'Create' and then 'Start Training' under the Trainings section.

Refer to the MNIST tutorial for an example of adding participants and initiating training.

## Online Inference

### Performing Inference with Trained Models

The frontend supports online inference:

1. **Choose a Model**: Select a trained model.
2. **Upload Data**: Submit an image file (JPEG, PNG) for analysis.
3. **Get Results**: The model processes the input and provides predictions or classifications.
