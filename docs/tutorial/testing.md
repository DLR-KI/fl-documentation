# Test Ecosystem Environment

In this section, we will briefly outline the process of setting up a test ecosystem environment.
Feel free to take a closer look at the individual scripts and files to better understand the entire process.
The goal of this section is to run a simple training.

## Requirements

This test ecosystem environment comes with some requirements:

- free system space
- access to all required projects
- [Docker](https://www.docker.com)
- [Docker Compose](https://docs.docker.com/compose)

## Environment Variables

The table below provides a list of available environment variables, complete with brief descriptions and their default
behaviors.
**Please be aware that some of these are compulsory and must always be defined!**

| Variable                                    | Description<br/>Default                                                      |
|:--------------------------------------------|:-----------------------------------------------------------------------------|
| `FL_DEMONSTRATOR_USERNAME` :exclamation:{ title="mandatory" } | Username of the client or participant required for server authentication.<br/>Default: N/A |
| `CLIENT_ID` :exclamation:{ title="mandatory" } | UUID of the client or participant required for server communication.<br/>Default: N/A |
| `S3_ENDPOINT` :material-exclamation-thick:{ title="mandatory for advanced tensorboard logging" } | The URL of the S3 instance.<br/>Default: N/A |
| `AWS_ACCESS_KEY_ID` :material-exclamation-thick:{ title="mandatory for advanced tensorboard logging" } | The S3 access key or username.<br/>Default: N/A |
| `AWS_SECRET_ACCESS_KEY` :material-exclamation-thick:{ title="mandatory for advanced tensorboard logging" } | The S3 access secret, for example, the password.<br/>Default: N/A |
| `FL_CLIENT_SETTINGS_MODULE`                 | Client settings module.<br/>Default: `dlr.fl.client.settings.Settings`       |
| `FL_CLIENT_ADDITIONAL_SYS_PATH`             | Additional import paths splitted via `:`.<br/>Default: empty                 |
| `FL_DEMONSTRATOR_BASE_URL`                  | Base URL of the FL Demonstrator server.<br/>Default: `http://localhost:8000` |
| `FL_DEMONSTRATOR_TRAINING_SCRIPT_EXECUTOR`  | Path to the script executor.<br/>Default: `python`                           |
| `FL_DEMONSTRATOR_TRAINING_SCRIPT_PATH`      | Path to the training script.<br/>Default: `src/main.py`                     |
| `FL_DEMONSTRATOR_TRAINING_WORKING_DIRETORY` | Working directory for the training script.<br/>Default: `.`                  |
| `FL_CLIENT_SERVER_HOST`                     | Client server hostname.<br/>Default: `0.0.0.0`                               |
| `FL_CLIENT_SERVER_PORT`                     | Client server port.<br/>Default: `8101`                                      |

Please note, some of these variables can be overridden within the [settings](./code-snippets.md#file-srcsettingspy)
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

## Get real

Once all the preparations are complete, we can finally initiate all required components, proceed with the entire process
of preparing the data and model, set up the server object, e.g. the user and training, and ultimately, commence the
actual training process.

```bash
# Navigate into the MNIST example client project
cd fl-demonstrator-mnist

# Docker cleanup
docker volume prune -f

# Download MNIST dataset and split it into 10 small and unique client subsets
python ./scripts/download-and-split.py

# Create a virtual demonstration network for all docker container
docker network create mnist-demo

# Start Federated Learning Platform
docker compose -f docker-compose.server.yml up -d --build

# Open Logs
# ! Start the following two command each in a seperated terminal session !
docker logs -f web
docker logs -f celery

# Create FL Demonstrator actor, clients and training
./scripts/create-participants.sh -f
./scripts/create-model.sh -f
./scripts/create-training.sh -f

# Build as well as start clients and be ready to train
docker compose --env-file ./responses/participants.env up -d --build
```

Now that everything is set up, let's begin the training.

```bash
# Start training via FL Demonstrator
./scripts/start-training.sh

# Optional
# See client logs (client-01)
docker logs -f client-01
```

Once the training is complete, and partially also during the training, you can see some model metrics in the web
frondend.
Just start a web frontend docker container and navigate to: <http://localhost:8080>.

```bash
# Start the web frontend
docker run -d --name frontend -e FL_DJANGO_SERVER_NAME=web:8000 --network mnist-demo -p 8080:8080 ghcr.io/dlr-ki/fl-demonstrator-frontend:main
docker logs -f frontend
```

After you finished your examinations, you can stop the `docker logs` commands by pressing ++ctrl+c++ or by closing their
terminal sessions.
Lastly, we need to shut down all components and tidy up.

```bash
# stop frontend
docker stop frontend
docker rm frontend

# stop all clients
docker compose down

# stop server components
docker compose -f docker-compose.server.yml down

# cleanup
docker network remove mnist-demo
```

/// details | Do all training steps at once
    type: tip
    open: True

Rather than manually executing all the commands to initiate all components, run the training, and clean up after the
preparations are complete, you can execute the `/scripts/all.sh` script.

If you're utilizing the `all.sh` script, please press ++ctrl+c++ after the training is finished.
If you opt to simply close or terminate the terminal running the `all.sh` script, the server and client components
won't be shut down and no final cleanup will be performed.

///
