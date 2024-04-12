# Test Ecosystem Environment

In this section, we will briefly outline the process of setting up a test ecosystem environment to train the MNIST models.

Therefore we will start 10 instances of the [fl-demonstrator-mnist](https://github.com/DLR-KI/fl-demonstrator-mnist) client in a docker-compose file.

In a real world scenario, each client would likely run on a separate machine. Other than that, the procedure described here can be very easily transformed to a real world setting.

## Requirements

This test ecosystem environment comes with some requirements:

- free system space
- access to all required projects
- [Docker](https://www.docker.com)
- [Docker Compose](https://docs.docker.com/compose)
- [jq](https://jqlang.github.io/jq)

## Environment Variables

The table below provides a list of available environment variables, complete with brief descriptions and their default
behaviors.
**Please be aware that some of these are compulsory and must always be defined!**

| Variable                                                                                                   | Description<br/>Default                                                                    |
| :--------------------------------------------------------------------------------------------------------- | :----------------------------------------------------------------------------------------- |
| `FL_DEMONSTRATOR_USERNAME` :exclamation:{ title="mandatory" }                                              | Username of the client or participant required for server authentication.<br/>Default: N/A |
| `CLIENT_ID` :exclamation:{ title="mandatory" }                                                             | UUID of the client or participant required for server communication.<br/>Default: N/A      |
| `S3_ENDPOINT` :material-exclamation-thick:{ title="mandatory for advanced tensorboard logging" }           | The URL of the S3 instance.<br/>Default: N/A                                               |
| `AWS_ACCESS_KEY_ID` :material-exclamation-thick:{ title="mandatory for advanced tensorboard logging" }     | The S3 access key or username.<br/>Default: N/A                                            |
| `AWS_SECRET_ACCESS_KEY` :material-exclamation-thick:{ title="mandatory for advanced tensorboard logging" } | The S3 access secret, for example, the password.<br/>Default: N/A                          |
| `FL_CLIENT_SETTINGS_MODULE`                                                                                | Client settings module.<br/>Default: `dlr.fl.client.settings.Settings`                     |
| `FL_CLIENT_ADDITIONAL_SYS_PATH`                                                                            | Additional import paths splitted via `:`.<br/>Default: empty                               |
| `FL_DEMONSTRATOR_BASE_URL`                                                                                 | Base URL of the FL Demonstrator server.<br/>Default: `http://localhost:8000`               |
| `FL_DEMONSTRATOR_TRAINING_SCRIPT_EXECUTOR`                                                                 | Path to the script executor.<br/>Default: `python`                                         |
| `FL_DEMONSTRATOR_TRAINING_SCRIPT_PATH`                                                                     | Path to the training script.<br/>Default: `src/main.py`                                    |
| `FL_DEMONSTRATOR_TRAINING_WORKING_DIRETORY`                                                                | Working directory for the training script.<br/>Default: `.`                                |
| `FL_CLIENT_SERVER_HOST`                                                                                    | Client server hostname.<br/>Default: `0.0.0.0`                                             |
| `FL_CLIENT_SERVER_PORT`                                                                                    | Client server port.<br/>Default: `8101`                                                    |

Please note, some of these variables can be overridden within the [settings](./fl-mnist-client.md#file-srcsettingspy)
module.

## Prepare dependencies

To run the whole ecosystem and start a actual dummy training we need to prepare all required components.
This include among others the server, its dependencies as well as the MNIST example client and its dependencies.

Lets create a designated folder inside our file system only for this demonstration purpose.
For all future steps we assume that we always begin in this new directory.

```bash
# create own folder and access it
mkdir fl-demo
cd fl-demo
```

Now, let's proceed with downloading all necessary components.

```bash
# MNIST example client project
git clone "https://github.com/DLR-KI/fl-demonstrator-mnist"
```

Make sure to install all package dependencies. More details [here](https://github.com/DLR-KI/fl-demonstrator-mnist/blob/main/README.md).

```bash
cd fl-demonstrator-mnist

# create virtual environment
virtualenv -p $(which python3.10) .venv
# or
# python -m venv .venv

# activate our virtual environment
source .venv/bin/activate

# update pip (optional)
python -m pip install -U pip

# install
./dev install -U -e ".[all]"

cd ..
```

After we download all components we can start to build them.
To avoid concerns about further dependencies and varying computer architectures, we'll start using Docker from here.

```bash
# pull server and base client components
docker pull ghcr.io/dlr-ki/fl-demonstrator-celery:main
docker pull ghcr.io/dlr-ki/fl-demonstrator-django:main
docker pull ghcr.io/dlr-ki/fl-demonstrator-client:main
docker pull ghcr.io/dlr-ki/fl-demonstrator-frontend:main

# build example client project Docker images
cd fl-demonstrator-mnist
docker compose build
cd ..
```
