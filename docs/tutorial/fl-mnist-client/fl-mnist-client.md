# The `fl-demonstrator-mnist` package

In this section, we describe the key components [`fl-demonstrator-mnist`][1] package.

Since the whole MNIST example client project code is open source, you can clone or fork it anytime.
If it captures your interest, don't hesitate to explore the code further.
You can even utilize the entire project as a template for your own future innovative projects.

## Files

The [`fl-demonstrator-mnist`][1] example client project contains many files.
Nevertheless, a few of these files hold greater significance and thus require additional consideration.
In essence, three or four primary files in this project serve as the core structure for your own training script:

1. `/src/settings.py` _(optional)_
2. `/src/main.py`
3. `/src/config.py`
4. `/src/training.py`

To test and manage the entire ecosystem along with its dependencies, you might find it beneficial to take a further
look into the `/scripts` folder.

Naturally, if you really want to understand anything and adapt anything perfectly to your needs, you will also need to
look into all other files.
But the previous mentioned files should be a good starting point.

### `/src/settings.py`

The settings file is completely optional.
It can be utilized to override the default configuration settings for the project.
If you choose to use it, you must appropriately set the `FL_CLIENT_SETTINGS_MODULE` environment variable.

Please note that these settings pertain to the project, such as the IP address and port on which the communication and
notification endpoint should listen, and not to the settings of the Machine Learning training itself.
Possible setting module variables are:

| Variable                                                                                                           | Description<br/>Default                                                                      |
| :----------------------------------------------------------------------------------------------------------------- | :------------------------------------------------------------------------------------------- |
| `FL_DEMONSTRATOR_BASE_URL`                                                                                         | Base URL of the FL Demonstrator server.<br/>Default: `http://localhost:8000`                 |
| `FL_DEMONSTRATOR_TRAINING_SCRIPT_EXECUTOR`                                                                         | Path to the script executor.<br/>Default: `python`                                           |
| `FL_DEMONSTRATOR_TRAINING_SCRIPT_PATH`                                                                             | Path to the training script.<br/>Default: `src/train.py`                                     |
| `FL_DEMONSTRATOR_TRAINING_WORKING_DIRETORY`                                                                        | Working directory for the training script.<br/>Default: `.`                                  |
| `SERVER_HOST` :material-information-outline:{ title="Deviating environment variable name: FL_CLIENT_SERVER_HOST" } | Client server hostname.<br/>Default: `0.0.0.0`                                               |
| `SERVER_PORT` :material-information-outline:{ title="Deviating environment variable name: FL_CLIENT_SERVER_PORT" } | Client server port.<br/>Default: `8101`                                                      |
| `MAIN_MODULE`                                                                                                      | Main entry point of the client server.<br/>Default: `dlr.fl.client.__main__.default_main`    |
| `COMMUNICATION_MODULE`                                                                                             | Communication module of the client.<br/>Default: `dlr.fl.client.communication.Communication` |

/// details | Pay attention to environment variables
    type: note
    open: True

Keep in mind that in most cases, the default behavior can be overridden by specific environment variables.
Also, if both the environment variable and the settings variable are set, the settings variable will be prioritized.

Also, be aware that not all settings variables can be accessed as environment variables.
A list of the supported environment variables can be found in the
[Environment Variables](./setup.md#environment-variables) section.

///

//// details | Example
    type: example
    open: True

Lets assume we want to setup some specific logging settings before the integrated web server, which act as communication
endpoint for the server, starts.
In this case we could inherit from `dlr.fl.client.settings.Settings`, create therefore our own Settings class and set a
specific `MAIN_MODULE`.
This main module could be a reference to our own self-coded function.
Now, we are free to do what we want.
In the following example we just initialize the logging behavior and continue with the default behavior.

Assume we wish to establish certain logging behaviors before the start of the integrated web server, which serves as the
communication and notification endpoint for the server.

In this scenario, we could inherit from `dlr.fl.client.settings.Settings`, thereby creating our own Settings class and
defining a specific `MAIN_MODULE`.
This main module could point to a function we've coded ourselves.
This gives us the freedom to do as we please.

In the subsequent example, we simply initialize the logging behavior and proceed with the default behavior:

```python title="src/settings.py"
from logging import getLogger

from dlr.fl.client.settings import Settings as SettingsBase
from dlr.ki.logging import load_default


def main() -> None:
    from dlr.fl.client.__main__ import default_main

    load_default("logs/train.log")
    getLogger("fl.client").info("logging initialized")
    default_main()


class Settings(SettingsBase):
    MAIN_MODULE: str = "settings.main"
```

/// details | Do not forget to enable the custom setting module
    type: note
    open: True

To activate the custom Settings class, you need to assign the `FL_CLIENT_SETTINGS_MODULE` environment variable to the
correct module path, which in this instance would be `src.settings.Settings`.

If you encounter difficulties in defining the correct path due to import issues stemming from SYS path settings, you
have the option to adjust them by setting the `FL_CLIENT_ADDITIONAL_SYS_PATH` environment variable.

///
////

### `/src/main.py`

The main script serves as the primary script file or the entry point for the training script.
It parses the provided training arguments `parse_args()`, establishes the main error handling, initiates the
communication between the client and server `main()`, and finally triggers the training `train()` or testing `test()`
process.

Fundamentally, as developers, our primary task is to modify the `USERNAME` and `PASSWORD` credentials in the `main()`
method.
While the `USERNAME` can also be configured via the `FL_DEMONSTRATOR_USERNAME` environment variable, the `PASSWORD` is
currently hard-coded.
You're welcome to modify this in line with your security specifications.

/// details | Credentials
    type: hint

```python  title="src/main.py" hl_lines="7 8"
def main(logger: Logger) -> None:
    ...
    logger.info(
        "Hint: FL_DEMONSTRATOR_USERNAME and CLIENT_ID must be set as environment variables"
        "and the later one must also be a valid UUID."
    )
    USERNAME = environ["FL_DEMONSTRATOR_USERNAME"]  # raise KeyError if not set
    PASSWORD = "mnist-secret"
    args = parse_args()
    ...
```

///

It's worth noting that the `parse_args()` method makes the script's call parameters readily identifiable.
Essentially, there's a single positional argument, the action, which can either be "train" or "test", along with a few
options:

- `--client-id CLIENT_ID`: This is the client UUID, which defaults to the `CLIENT_ID` environment variable.
- `--training-id TRAINING_ID`: This is the training UUID, and this argument is mandatory.
- `--round ROUND`: This represents the training round, and this argument is compulsory.
- `--model-id MODEL_ID`: This is the global model UUID, and this argument is obligatory.

/// details | Help text

```txt
main.py [action] [options]

MNIST example main.py

positional arguments:
{train,test}          Action to perform

options:
-h, --help            show this help message and exit
--client-id CLIENT_ID
                      Client UUID
--training-id TRAINING_ID
                      Training UUID
--round ROUND         Training round
--model-id MODEL_ID   Global model UUID
```

///

All other methods should exhibit a robust default behavior.
Even the `train()` method for training or the `test()` method for testing do not necessarily require modification, as
the actual training configurations, along with the training and testing processes themselves, are managed in
`/src/config.py` and `/src/trainings.py`.

### `/src/config.py`

The trainings configuration file contains the `Config` class which contains most of the settings which are interesting
in a Machine Learning training.
This includes the number of local training epochs, the learning rate as well as the chosen optimizer and scheduler and
many more.

The complete set of adjustable parameters includes:

- `device`: By default, this is set to `cuda` if CUDA is available, otherwise, it falls back to `cpu`.

- `epochs`: This represents the number of local training epochs, with the default set to `3`.

- `batch_size`: The batch size is a hyperparameter that defines the number of data samples propagated through the
    network before updating the internal model parameters.
    It impacts the model's learning speed and quality.
    This is set to `8` by default.

- `learning_rate`: This determines the step size at each iteration while moving toward a minimum of a loss function.
    It decides how quickly or slowly we want to update the model parameters.
    Please keep in mind, smaller values can lead to a slower learning speed, while larger values might result in
    unpredictable behavior during the training.
    The default value is set to  `0.1`.

- `optimizer`: The optimizer is responsible for adjusting the model parameters, such as weights and learning rate, in
    each iteration to minimize the model error.
    In simpler terms, it modifies your model's attributes to reduce losses, thereby accelerating the results.
    [Stochastic Gradient Descent (SGD)][9] and [Adam][8] are commonly used optimizers.
    By default, [Adadelta][7], an extension of SGD, is used.

    For more information on optimizers and to explore various other types, visit:
    <https://pytorch.org/docs/stable/optim.html>.

- `scheduler`: The learning rate scheduler, is a tool that adjusts the learning rate during the training process.
    The purpose of a scheduler is to change the learning rate over time, usually to decrease it.
    The idea is to start with a high learning rate to progress quickly, and then decrease it over time to allow the
    model to settle into the optimal solution.
    There are various strategies for this, such as step decay, like [`StepLR`][6] which is also set as default, where
    the learning rate is reduced by a factor every few epochs, or exponential decay, where the learning rate decreases
    exponentially over time.

    If you're interested in learning more about adjusting the learning rate or exploring various learning rate
    schedulers, please refer to the PyTorch documentation:
    <https://pytorch.org/docs/stable/optim.html#how-to-adjust-learning-rate>.

    Note, if you choose a scheduler which can not be initialize with
    `scheduler(optimizer, step_size=1, gamma=config.gamma)` please adjust the code inside the `train()` method of the
    `/src/main.py` file.

- `gamma`: This is often used in learning rate decay schedules.
    For instance, in the case that the `scheduler` is set to `StepLR`, `gamma` is the factor by which the learning rate
    gets multiplied at each step.
    If the `gamma` is set to `0.7`, which is the case by default, then the learning rate will be multiplied by 0.7 at
    each step, effectively reducing the learning rate over time.
    This can help the model to converge more effectively during training by making smaller adjustments as it gets closer
    to the optimal solution.

- `loss`: The loss function quantifies the disparity between the predicted result and the actual value, and it's this
    function that we aim to minimize during training.
    We compute the loss by making a prediction using the inputs from our given data sample and contrasting it with the
    true data label value.

    Frequently used loss functions include [Mean Square Error (MSELoss)][11] for regression tasks, and
    [Negative Log Likelihood (NLLLoss)][12] for classification tasks.
    The default [`CrossEntropyLoss`][10] is also a commonly used lossfunction which combines [LogSoftmax][13] and
    NLLLoss.

    A list of loss function offered by PyTorch can be visite at
    <https://pytorch.org/docs/stable/nn.html#loss-functions>.

- `logger`: The logger, obtained from `logging.getLogger("fl.client")`, is the default.

- `log_interval`: The log interval is set to `20` by default.

<!-- invisible header - only for linking purposes -->
<h4 id="logging"></h4>

The `Config` class is also responsible for managing the logging object handle, implying that it oversees file or stream
resources such as the `SummaryWriter` used for logging purposes.

/// details | Usage example
    type: example

```python
with Config(args) as config:
    trained_model, metrics, sample_size = train(model, config)
    # ...
```

///

As a standard, this sample project utilizes decentralized tensorboard logging.
This necessitates an S3 endpoint and the appropriate credentials.
The necessary information can be provided via environment variables:

- `S3_ENDPOINT`: The URL of the S3 instance
- `AWS_ACCESS_KEY_ID`: The access key or username
- `AWS_SECRET_ACCESS_KEY`: The access secret, for example, the password

Additionally, the sophisticated tensorboard logging anticipates an S3 bucket named `trainings`.
Subsequently, the log will be recorded at `s3://trainings/<TRAINING_ID>/<CLIENT_ID>`.
A convenient method to establish a suitable environment is to set up a MinIO server that contains a bucket named
`trainings`.

If you find this process a bit too complex and wish to simplify it, you could opt to disable the decentralized
tensorboard logging and only log locally instead.
To accomplish this, simply modify the `log_dir` of the `SummaryWriter` within the constructor of the `Config` class in
the `src/config.py` script.
Also, it's strongly advised to log to a volume when your clients are docker containers.
As an example, you could initialize the summary writer as
`#!python SummaryWriter(f"runs/{self.args.training_id}/{self.args.client_id}")` and define the `runs` directory as
docker volume.

/// details | `SummaryWriter` code changes to disable decentralized tensorboard logging
    type: example

```diff title="src/config.py"
class Config(ContextDecorator):
    ...
    def __init__(self, args: Namespace) -> None:
        self.args = args
-       self.summary_writer = SummaryWriter(f"s3://trainings/{self.args.training_id}/{self.args.client_id}")
+       self.summary_writer = SummaryWriter(f"runs/{self.args.training_id}/{self.args.client_id}")
    ...
```

///

### `/src/training.py`

Assuming that the self-developed client code is entirely based on this client example, this file could be the most vital
for each client developer.
It manages the actual training and testing, along with defining and loading the training and test data.

In total there are four methods:

- `get_datasets()`
- `get_data_loader()`
- `train_epoch()`
- `test()`

#### `get_datasets()`

In essence, this method's function is to load and return the training and testing datasets.

In this example project, a Python [pickle][2] file is loaded from `./data/client-train.pt` and `./data/client-test.pt`
relative to the root project.
This file contains a predefined [Dataset][3] from PyTorch.

This method can be modified as per your requirements.
An example of how to generate such pickle files, utilizing the [MNIST][4] dataset with the assistance of the torchvision
library, can be seen in the script `/scripts/download-and-split.py`.
This script downloads the dataset (both training and test sets), restricts the size of the test dataset, and partitions
the training dataset into 10 distinct sections of varying sizes.
This is merely for testing and demonstration purposes.
In a real life scenario, each client will possess its own private data that it needs to supply.

It's noteworthy that Torchvision provides a variety of built-in datasets, a comprehensive list of which can be accessed
here: <https://pytorch.org/vision/stable/datasets.html>.
You're welcome to experiment with several other datasets.
However, please keep in mind the potential requirements for the model.

A simple example of creating a model for future upload and use is available in the script
`/scripts/create-torch-model-file.py`.
This script is straightforward and easy to comprehend.
It merely defines a PyTorch model with several layers and saves it as a pickle file on the local file system.
This model file can subsequently be uploaded as a common model foundation or the initial "global" model to the server.

#### `get_data_loader()`

The purpose of this method is to retrieve the data loaders for training and testing.
The content is quite straightforward.
It merely transforms the training and test datasets that it obtains from the `get_datasets()` method.

#### `train_epoch()`

This method is responsible for executing the training of a model for a single epoch.

If you're new to machine learning and its development, we suggest going through the entire [PyTorch Basic Tutorial][5].
A comprehensive example of the `train_epoch()` method, the subsequent `test()` method, as well as the `train()` method
from the `/src/main.py` file, can be found at this section of the tutorial:
<https://pytorch.org/tutorials/beginner/basics/optimization_tutorial.html#full-implementation>.

/// details | Just one single epoch!
    type: important
    open: True

It's crucial to note that this method trains the model for just one single epoch.
The loop for the training epochs in this sample project is located in the `train()` method within the `/src/main.py`
file by default.

///

#### `test()`

The purpose of this method is to conduct tests on a model.
It evaluates how well a model performs based on your test data.
You may consider calculating and presenting other or additional metrics.
As a standard, this example project calculates the following metrics:

- **Loss**: In machine learning, the loss is a measure of the discrepancy between the model's predictions and the actual
    data.
    It's a function that the model seeks to minimize during training.
- **Accuracy**: Accuracy is a metric that measures the proportion of correct predictions made by the model out of all
    predictions.
    It's often used in classification problems.
- **AUROC (Area Under the Receiver Operating Characteristic)**: AUROC is a performance measurement for classification
    problems.
    It tells how much a model is capable of distinguishing between classes.
    The higher the AUROC, the better the model is at predicting 0s as 0s and 1s as 1s.
- **Recall**: Also known as sensitivity or true positive rate, recall is the proportion of actual positive cases that
    the model correctly identified as positive.
    It's particularly important in cases where false negatives are more costly than false positives.
- **Precision**: Precision is the proportion of positive identifications that were actually correct.
    It's important in situations where false positives are more costly than false negatives.

[1]: https://github.com/DLR-KI/fl-demonstrator-mnist
[2]: https://docs.python.org/3/library/pickle.html  
[3]: https://pytorch.org/docs/stable/data.html#torch.utils.data.Dataset
[4]: http://yann.lecun.com/exdb/mnist
[5]: https://pytorch.org/tutorials/beginner/basics/intro.html
[6]: https://pytorch.org/docs/stable/generated/torch.optim.lr_scheduler.StepLR.html#torch.optim.lr_scheduler.StepLR
[7]: https://pytorch.org/docs/stable/generated/torch.optim.Adadelta.html
[8]: https://pytorch.org/docs/stable/generated/torch.optim.Adam.html
[9]: https://pytorch.org/docs/stable/generated/torch.optim.SGD.html
[10]: https://pytorch.org/docs/stable/generated/torch.nn.CrossEntropyLoss.html
[11]: https://pytorch.org/docs/stable/generated/torch.nn.MSELoss.html
[12]: https://pytorch.org/docs/stable/generated/torch.nn.NLLLoss.html
[13]: https://pytorch.org/docs/stable/generated/torch.nn.LogSoftmax.html
