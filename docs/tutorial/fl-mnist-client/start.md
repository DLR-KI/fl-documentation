<!--
SPDX-FileCopyrightText: 2024 Benedikt Franke <benedikt.franke@dlr.de>
SPDX-FileCopyrightText: 2024 Florian Heinrich <florian.heinrich@dlr.de>

SPDX-License-Identifier: CC-BY-4.0
-->

# Get real

Once all the preparations are complete, we can finally initiate all required components, proceed with the entire process
of preparing the data and model, set up the server object, e.g. the user and training, and ultimately, commence the
actual training process.

Rather than manually executing all the commands to initiate all components, run the training, and clean up after the
preparations are complete, you can execute the `/scripts/all.sh` script.

If you're utilizing the `all.sh` script, please press ++ctrl+c++ after the training is finished.
If you opt to simply close or terminate the terminal running the `all.sh` script, the server and client components
won't be shut down and no final cleanup will be performed.

/// details | Hint
    type: hint
    open: True

If we get problems and want to start again, please make sure to clean up first.
See the [cleanup section](#clean-up) below.

///

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
docker compose -f docker-compose.server.yml up -d

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

If you are starting everything for the first time and didn't disable the MinIO logging which is futher described
[here](./fl-mnist-client.md#logging),
lets look into MinIO for a bit before we finally begin the training.
By default, you should now be able to visit your local MinIO instance: <http://localhost:9001>.
The preset username and password are `admin` and `password` which can be changed inside the `/server.env` file of the MNIST example client project.
After your successful login, you should now be able to manage your MinIO instance.
In our case we would like to navigate to _Administrator > Buckets_ and create a new bucket names `trainings`.
This bucket will later be used to store all kind of debug and logging information which are written to the tensorboard.
It is a quite useful setup at least during the development stage.

/// details | Create MinIO Bucket via terminal
    type: tip
    open: True

Instead of using the MinIO web interface, you can also create the `trainings` bucket via the terminal.
The easiest method is to use the MinIO client `mc`, which can be installed locally or used from a designated Docker container.

To use the following command from inside the designated Docker container, start its terminal with:

```bash
docker run -it --entrypoint /bin/bash minio/mc
```

Then, create the bucket using the MinIO client:

```bash
# create an alias 'myminio' with your server URL and credentials
mc alias set myminio http://localhost:9000 admin password
# create the bucket 'trainings'
mc mb myminio/trainings
```

///

Now everything is set up.
Lets finally start the training.

```bash
# Start training via FL Demonstrator
./scripts/start-training.sh

# Optional
# See client logs (client-01)
docker logs -f client-01
```

Once the training is complete, and partially also during the training, you can see some model metrics in the web
frontend.
Therefore, let us start the web frontend docker container.

```bash
# Start the web frontend
docker run -d --name frontend -e FL_DJANGO_SERVER_NAME=web:8000 --network mnist-demo -p 8080:8080 ghcr.io/dlr-ki/fl-demonstrator-frontend:main
docker logs -f frontend
```

Then you can navigate to: <http://localhost:8080>.

The example trainings actor user credentials are:

- username: `mnist-client-01`
- password: `mnist-secret`

Following this pattern, other participants, which are only clients and no actors, can log in with usernames `mnist-client-02` to `mnist-client-10`, all using `mnist-secret` as the password.

<!-- invisible header - only for linking purposes -->
<h2 id="clean-up"></h2>

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

/// details | That is it - Congratulations!
    type: tip
    open: True

You have now successfully trained your first model with federated learning. If you now want to dig even deeper in the federated learning universe, we recommend you to read the next section. There we describe the components of the `fl-demonstrator-mnist` package you used in this example.

///
