<!--
SPDX-FileCopyrightText: 2024 Benedikt Franke <benedikt.franke@dlr.de>
SPDX-FileCopyrightText: 2024 Florian Heinrich <florian.heinrich@dlr.de>

SPDX-License-Identifier: CC-BY-4.0
-->

# Getting Started

In the following tutorial, you will create a simple project to train models based on the popular [MNIST][1] dataset.
This dataset is a large collection of handwritten digits and is widely used in the field of Machine learning and image
processing.
It provides a solid foundation for training and testing Machine learning models, particularly those involved in image
recognition tasks.

The tutorial project is written in Python, a versatile and widely-used programming language that is particularly strong
in the field of Machine learning.
Python's extensive library support and easy-to-read syntax make it an excellent choice for developing Machine learning
clients.

The source code on which the tutorial is based for the process of training a neural network is taken from
[PyTorch's traditional Machine Learning MNIST example][2].
[PyTorch][3] is a powerful open-source Machine learning library for Python, and its MNIST example is a well-established
benchmark in the field.
However, it's important to note that this example does not involve Federated Learning.
Hence, you will create it :winking_face:.

To assist you in this process, the DLR Federated Learning Ecosystem provides a small Python package:
[`fl-demonstrator-mnist`][5].
This package implements tools and algorithms to train models based on the MNIST database.

It uses the [`fl-demonstrator-client`][4] package as a dependency which facilities all the client-server communication. In the chapter [Create your own client](../your-own-client/getting-started.md) you will use this package to create your own federated learning client.

Both packages are open source and can be found on Github.

[1]: http://yann.lecun.com/exdb/mnist
[2]: https://github.com/pytorch/examples/blob/55c663f9a50f79cb6a905c56d886343c555d34a9/mnist/main.py
[3]: https://pytorch.org
[4]: https://github.com/DLR-KI/fl-demonstrator-client
[5]: https://github.com/DLR-KI/fl-demonstrator-mnist
