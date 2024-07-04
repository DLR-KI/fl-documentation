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

# Model Aggregation Strategies

## Synopsis

Quick guide to choose the right strategy:

- Homogeneous data & simple requirements: choose [**FedAvg**](#1-fedavg-federated-averaging).
- Heterogeneous data or diverse client resources: choose [**FedProx**](#2-fedprox-federated-proximal).
- Data scarcity or strong data imbalance among clients: choose [**FedDC**](#3-feddc-federated-daisy-chaining).

If unsure, FedProx is usually a reasonable choice.
FedAvg is the simplest method and can serve as a baseline as well as for testing purposes.

## Introduction

This guide provides an overview of the available model aggregation strategies in our federated machine learning
platform.
It is designed to help both non-experts and experts in machine learning understand the nuances of each method and
choose the appropriate strategy for their specific use case.

### Understanding Federated Learning

Before explaining the various aggregation strategies, it is essential to grasp the concept of federated learning.
Federated learning is a machine learning approach where a model is trained across multiple decentralized
devices (clients) holding local data samples, without exchanging them.
This method helps in preserving data privacy and reducing the need for central data storage.

#### Key Advantages

- Privacy Preservation: Data remains on the client's device, reducing privacy risks.
- Reduced Data Centralization: Limits the risks associated with central data repositories.
- Efficiency in Bandwidth: Reduces the need for high bandwidth to transfer large datasets.

#### Need for dedicated aggregation strategies

In federated learning, model aggregation is a crucial step, because models trained on local client data need to be
combined (aggregated) in order to produce a more robust and reliable global model.
The global model is subsequently transferred to the clients and then used as a new local model on which training
continues.
However, due to the diversity of the applications and client environments, there is no single aggregation strategy
that performs well in all cases.

The key challenges for finding the proper aggregation strategy are:

- **Data Heterogeneity**: In real-world scenarios, data across clients can vary significantly (e.g., distribution,
    volume, and quality).
    Such discrepancies can cause local models to converge to different local optima, which may not represent the global
    optimum accurately.
- **Model Drift**: As a result of data heterogeneity, when local models are trained independently on disparate datasets,
    they may diverge significantly from the global model.
    This divergence, known as "model drift", can lead to suboptimal performance when the local models are aggregated.
- **Impact on Global Model**: The aggregated global model might not capture the true underlying pattern of the entire
    dataset, particularly when the data distribution extremely heterogenous across clients.

#### Further literature

- General overview on federated learning:
    [Wang et al (2021) "A field guide to federated optimization"][1]
- On aggregation strategies:
    [Moshawrab et al (2023) "Reviewing Federated Learning Aggregation Algorithms; Strategies, Contributions, Limitations and Future Perspectives"][2]
- On effects of data heterogeneity:
    [Li, Diao, Chen, He (2022) "Federated learning on non-iid data silos: An experimental study"][3]

### Real-Life Scenarios in the Automotive Sector

Federated learning can be instrumental in the automotive sector.
For instance, self-driving cars can use it to improve their algorithms for navigation and obstacle detection without
sharing raw sensor data, thus preserving privacy and reducing data transfer needs.
Similarly, federated learning can optimize predictive maintenance in fleets of trucks by learning from each vehicle's
unique operating conditions and maintenance history.
These scenarios emphasize the need for efficient, secure, and adaptable model aggregation methods in federated learning
systems, particularly in environments where data privacy and minimal data transmission are crucial.

## Model Aggregation Strategies

The following the three strategies are available on our platform:

### 1. FedAvg (Federated Averaging)

#### Overview

- FedAvg is the simplest and most widely used aggregation strategy.
- It involves training local models on client devices and then averaging their parameters to update the global
    model.
- It can serve as a baseline against which the performance of more sophisticated methods can be compared.

#### Best Suited For

- Scenarios with relatively homogeneous data distribution across clients.
- Scenarios where simplicity and computational efficiency are priorities.
- However, even for heterogenous data distributions, FedAvg can perform surprisingly well and should thus be given
    a try.

#### Technical Details

- The training procedure consists of the following steps:
    1. Each client trains the model on their local data.
    2. The model parameters (weights, biases) from each client are periodically sent to a central server and are
        averaged to form a single global model.
    3. The global model is then sent to each client and replaces their local models.
- The approach does not sufficiently address the issue of model drift caused by heterogeneous client data and
    capabilities.
    Besides this shortcoming, the resulting model accuracy is reasonably (and even surprisingly) good.
- Benchmarks:
    - Centralized training baseline: a ResNet18-neural network model trained (in a non-federated way) on the image
        dataset CIFAR10 typically achieves up to 92% test accuracy
    - The same model under federated training with FedAvg achieves around 80%-90% test accuracy (the longer the
        local training time interval, the higher accuracy).
        For strong data heterogeneity or scarcity, accuracy can drop to 50% or even lower.
- Reference: [McMahan et al (2017) "Communication-efficient learning of deep networks from decentralized data"][4]

### 2. FedProx (Federated Proximal)

#### Overview

- FedProx improves FedAvg by dealing with data heterogeneity and challenges like varying computation and communication
    capabilities of the clients.

#### Best Suited For

- Environments where the available data among the clients varies in number, quality, and structure.
- Situations where clients have varying computational resources and thus do not all communicate with the central server
    at the same rate.

#### Technical Details

- Similar to FedAvg in aggregating model parameter updates, i.e., the code executed at the central server as well as
    the communication steps between server and clients are the same as for FedAvg.
- The difference lies in how the local training is performed: In FedProx, a regularization term is incorporated into
    the local model training that ensures that each local model remains relatively close to the global model.
    This prevents large model drifts due to client data heterogeneity, which otherwise might infect the global model
    and thus the whole system.
- Compared to FedAvg, the increased computational demand is very mild, but the accuracy gain is significant.
- Benchmarks: under the same conditions as FedAvg (see above), FedProx typically achieves 1-3% higher test accuracies.
    Under strong data imbalance or scarcity, accuracies can drop to 50%.
- Reference: [Li et al (2020) "Federated optimization in heterogeneous networks"][5]

### 3. FedDC (Federated Daisy-Chaining)

#### Overview

- FedDC is tailored for scenarios with data scarcity. It involves circulating models among clients for training before
    aggregation.
- The method can be combined with various aggregation strategies such as FedAvg and FedProx.

#### Best Suited For

- Cases where data is scarce or highly imbalanced among clients.
- Situations requiring enhanced model robustness in data-sparse environments.

#### Technical Details

- The method ensures a more diverse training experience for the model before the averaging step.
    Technically, this prevents overfitting of models to the data, which otherwise would lead to limited robustness and
    generalization ability of the global model.
- The training steps are similar to FedAvg as described above, except that, at selected times, the models are circulated
    among the clients instead of being sent to the central server.
    After the circulation step, the models are continued to be trained at their new clients, such that the circulated
    model effectively "sees" more data than available at its original client.
- The circulation step can be repeated a number of times before finally the local models are sent to the central server
    for aggregation.
    The actual model aggregation can be performed using any desired strategy, such as FedAvg or FedProx.

- After aggregation, the global model is returned to the clients, which continue their local training until another
    circulation or aggregation round begins.
- Benchmarks: For strong data imbalance among clients, FedDC still achieves around 63% test accuracy
    (whereas FedAvg/FedProx achieve ~50% in such cases).
    For mild data imbalance, FedDC always improves upon Fed/FedProx alone, although with increasingly smaller gain.
- Reference: [Kamp, Fischer, Vreeken (2022) "FedDC: Federated Learning from Small Datasets"][6]

[1]: https://arxiv.org/abs/2107.06917
[2]: https://doi.org/10.3390/electronics12102287
[3]: https://ieeexplore.ieee.org/document/9835537
[4]: https://arxiv.org/abs/1602.05629
[5]: https://proceedings.mlsys.org/paper_files/paper/2020/hash/1f5fe83998a09396ebe6477d9475ba0c-Abstract.html
[6]: https://arxiv.org/abs/2110.03469

<!-- cspell:ignore Diao Kamp McMahan Moshawrab Vreeken -->
