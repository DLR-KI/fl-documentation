# Components and Iteration

![DLR Federated Learning Ecosystem](../../assets/tutorial/ecosystem.drawio.png)

The DLR Federated Learning ecosystem you will work on involves two main components: the server and the clients.
Communication between these components is crucial, especially for the exchange of model and metadata.
An independent component of this setup is the statistics web server, which simply provides model statistics and other
useful information.

## Server

The server is implemented in the [`fl-demonstrator`](https://github.com/DLR-KI/fl-demonstrator) package. The server in this architecture is mainly fixed.

## Clients

The client-side training is flexible.
You have the freedom to create and shape your own client training, tailoring it to your specific needs and objectives.
This flexibility allows for a more personalized and efficient approach to Machine learning.

There are three approaches to implement the clients which are increasingly more flexible but also increasingly more complex.

### First Approach

Use the [`fl-demonstrator-mnist`][1] package as a client. This package is specifically tailored to train a model based on the popular [MNIST][1] dataset. The procedure to set up a test environment with this package is described in the next chapter.

| difficulty | flexibility |
| ---------- | ----------- |
| easy       | low         |

### Second Approach

Use the [`fl-demonstrator-client`][2] package to create your own client. You can use it to train any model you want. It is almost like writing code for traditional machine learning, since the `fl-demonstrator-client` does most of the "federated" part for you. The procedure to set up a test environment with this package is described in chapter [3](../your-own-client/getting-started.md).

| difficulty | flexibility |
| ---------- | ----------- |
| medium     | medium      |

### Third Approach

Create your own web server or endpoint and handle all the communication yourself.

| difficulty | flexibility |
| ---------- | ----------- |
| high       | high        |

[1]: http://yann.lecun.com/exdb/mnist
[2]: https://github.com/DLR-KI/fl-demonstrator-client
