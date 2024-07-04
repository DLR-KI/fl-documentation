---
# as long the page is not splittet into multiple files/sections
hide:
  - navigation
---
<!--
SPDX-FileCopyrightText: 2024 Benedikt Franke <benedikt.franke@dlr.de>
SPDX-FileCopyrightText: 2024 Florian Heinrich <florian.heinrich@dlr.de>

SPDX-License-Identifier: CC-BY-4.0
-->

# Uncertainty Quantification Methods

## Synopsis

Quick guide to choosing the right strategy for uncertainty quantification:

- Robust estimates for complex problems: choose [**Ensembling**](#1-ensembling).
- Limited resources and rapid predictions: choose [**MC Dropout**](#2-monte-carlo-dropout).
- Heterogenous data is available and a detailed uncertainty quantification is required: choose [**SWAG**](#3-stochastic-weight-averaging-gaussian-swag).

## Introduction

This guide provides an overview of the available methods for **uncertainty quantification (UQ)** in our federated machine learning platform.

### Why are UQ methods necessary?

- Uncertainty is naturally associated with data (e.g., variation in the lifetimes of machine parts) as well as with choice of the model (e.g., choosing an overly simple model typically leads to a larger prediction error).
  Standard machine learning methods are not designed to explicitly capture such uncertainties.
  Instead, their output is presented without a reliable confidence level.
  Accordingly, additional techniques need to be incorporated into the machine learning pipeline to correctly estimate the uncertainty of the predictions.

### How do UQ methods work?

- The UQ methods available here fundamentally work all in the same way: instead of just a single prediction, a range of predictions are generated.
  These are averaged to form a **mean**, which serves as the actual prediction replacing the single output of the original model.
  The associated **variance** accordingly quantifies the uncertainty of the prediction.
  This approach is akin to consulting multiple experts instead of relying on a single opinion.
- The UQ methods used here have originally been developed for centralized machine learning models and later adapted to a federated setting.
  They represent the state of the art and have comparable performance for typical benchmark datasets.
  For further information, see Linser et al [_"Approaches to uncertainty quantification in federated deep learning"_](https://rdcu.be/dyHwF) (2021).

### What is the use of UQ?

- Real-world data typically shows some intrinsic variability (e.g., lifetimes of parts) or contains errors (e.g., wrong labels).
  A machine learning model enhanced by a UQ method represents these variabilities in a reliable way.
- Providing an uncertainty estimate alongside predictions enhances the trustworthiness of a model and makes it more robust against unforeseen anomalies.
- UQ can reveal areas where a model is underperforming or lacks data, guiding further model development and requirements for data collection.
- A positive side effect of a UQ method is that the predictions of the underlying machine learning model typically become more robust against overfitting (i.e., undesired memorization of irrelevant details in the data).

## Methods for Uncertainty Quantification

The following three strategies are available on our platform:

### 1. Ensembling

#### Overview

Ensembling involves training multiple machine learning models independently and then aggregating their predictions.
This approach utilizes the intrinsic diversity among trained models to obtain the prediction uncertainty.

#### Use cases

- **Robust error estimation**: Useful in scenarios where a robust measure of confidence in predictions is crucial and computational resources are not constrained.
- **Complex datasets or models**: Performs well on heterogenous datasets and complex machine learning models (only constrained by computational resources, see below).

#### Caveats

- **Resource intense**: For very high-dimensional problems or complex architectures, the computational and memory requirements for Ensembling can become a limiting factor.
- Ensembling does not work reliably if the model is too simple, such as linear regression (for details see below).

#### Technical Details

- **Implementation**: Several independent machine learning models are trained, each starting from a different initialization.
  These different random initializations result in a diverse set of models, which need to be stored for later prediction generation.
- **Prediction aggregation**: Predictions from individual models are combined to produce a final averaged prediction and uncertainty estimate.
   By aggregating multiple models, the Ensembling also prevents overfitting and reduces the overall prediction error.
- **Caveat**: In order for ensembling to work, the trained model must be sufficiently complex such that its final state still depends somewhat on the initialization.
  This is usually the case for typical nonlinear neural network models, especially if the model is overparameterized.
  However, it is usually not the case for linear regression problems, since the final solution obtained after training is independent of the initialization.
  Accordingly, there would be no variability between the trained models and therefore averaging would be meaningless.
  In such a case, either an alternative UQ method must be used, or one may try to train the model on different sub-samples of the data.
- Although Ensembling has no rigorous theoretical underpinning, it usually performs surprisingly well and it considered state-of-the art to obtain robust uncertainty estimates for complex problems.
- **Resource requirements**: Ensembling demands relatively high computational and memory resources due to the need to train and store multiple models.

### 2. Monte-Carlo Dropout

#### Overview

- Monte-Carlo (MC) Dropout is a technique which modifies the architecture of a trained model such that its predictions vary each time a prediction is made, providing a basis for uncertainty estimation.
  MC Dropout offers a balance between performance and quality, with minimal modifications of the model.

#### Use cases

- **Rapid deployment**: Compared to other methods, MC Dropout works on already trained models with minimal modifications of the architecture.
- **Resource-constrained environments**: MC Dropout is less resource-intensive compared to Ensembling or SWAG.
  It is thus especially suitable for scenarios that require immediate uncertainty estimation.

#### Caveats

- The amount of dropout is a model parameter needs to be chosen beforehand by the user.
  Commonly used values are expected to work well with typical models and datasets in a federated setting.  

#### Technical Details

- **Implementation**: Dropout layers are added to a trained neural network model and are randomly activated during the prediction (inference) phase.
  Effectively, the network uses a different sub-network for each prediction.
- **Prediction aggregation**: To obtain an uncertainty estimate, the network is run multiple times with dropout to generate a distribution of outputs.
  These outputs are averaged to form a single estimate, while the variance is used as a measure of uncertainty.
- **Caveats**: Monte-Carlo Dropout offers a balance between computational efficiency and accuracy of uncertainty estimation, making it suitable for real-time applications.
  However, the level of dropout needs to be calibrated on a test dataset in order to not excessively distort the original network’s performance.
  In practice, values between 0.2 and 0.5 are used for the amount of dropout and are expected to work in typical scenarios.
- **Note**: MC Dropout is to be distinguished from the dropout method occasionally used during training in order to reduce overfitting.
  Both methods can be employed in the same model though.

### 3. Stochastic Weight Averaging Gaussian (SWAG)

#### Overview

- SWAG is a method to quantify uncertainty in complex neural networks based on a statistical model of their internal parameters.

#### Use cases

- Suitable if a large and diverse dataset is available for training.
- SWAG has an established theoretical underpinning in terms of Bayesian statistics, making it useful if a detailed understanding of model uncertainty is required.

#### Caveats

- If the amount of data is very limited or the data is very homogeneous, SWAG might be less reliable compared to other methods.
- SWAG requires additional storage and communication in a federated setting (although less than Ensembling).
  Hence, if computational resources are constrained, methods like MC Dropout might be preferable.
- Requires a certain understanding of the training dynamics in order to correctly employ it and judge its outputs.

#### Technical Details

- **Implementation**: Snapshots of the model parameters (neural network weights) are collected at regular intervals during the later stages of training of each client model ("stochastic weight averaging").
  The weight snapshots are collected among the clients and used to determine a Gaussian distribution, which reflects the range of plausible weight configurations the global model might adopt, consistent with the data.
- **Prediction aggregation**: By sampling from this weight distribution, different global models are generated, each generating slightly different predictions.
  The variability among these predictions serves as an indicator of uncertainty.
- SWAG captures the uncertainty of a model in terms of a statistical distribution of its parameters (posterior distribution) and thus entails more information than methods like Ensembling or MC Dropout, which are based on point estimates.
  SWAG therefore allows in principle for a detailed understanding of model uncertainty in terms of Bayesian statistics.
- **Caveats**: SWAG relies on data variations observed during model training.
  In scenarios with where the number of data samples is much less than the number of neural network weights (i.e., overparameterized neural networks) or where the data is very homogeneous, the model might not explore a diverse set of solutions, making SWAG less effective.
- **Computational considerations**: SWAG is more computationally intensive than MC Dropout, but less so compared to Ensembling, since only a single model per client needs to be trained.
  Rigorous application of SWAG requires to reach a stationary regime in the training process (where the loss is small and remains almost constant), which can be time consuming for large (overparameterized) models, especially in a federated setting.
  In particular, the intervals between the federated aggregation steps must be sufficiently large to allow the stationary training regime to develop.
