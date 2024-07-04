---
hide:
    - navigation
---
<!--
SPDX-FileCopyrightText: 2024 Benedikt Franke <benedikt.franke@dlr.de>
SPDX-FileCopyrightText: 2024 Florian Heinrich <florian.heinrich@dlr.de>

SPDX-License-Identifier: CC-BY-4.0
-->

# Architecture

Welcome to the architectural documentation for the FL platform.
This document serves as a high-level introduction to the architecture underpinning our advanced platform, designed to provide insight into how we've structured our system to meet our goals of scalability, performance, and reliability.
This overview will guide you through the main components, their interactions, and the key design decisions that shape our system.

## Key Points

DLR Federated Learning platform is:

- executable on the majority of Linux systems
- implemented in Python (>=3.10)
- providable via Docker container
- compatible to the AI library PyTorch

## Technical Context

On a high level, the FL platform consists of a web API, a task queue for the AI and a database for persistence.
The Web-API can also be used via single page fronted application.

![building-block-view](../assets/architecture/BuildingBlock.drawio.png "Building Block View")

Inference can occur online, enabling predictions directly from a server-hosted model without downloading it to the client.
This method ensures seamless predictions and accommodates clients with varying computational capacities, streamlining accessibility and efficiency.

![online-inference](../assets/architecture/online-inference.drawio.png "Online Inference")

Alternatively, clients can download the model for offline usage.
This option provides flexibility, allowing users to access predictive capabilities without transfering the data, enhancing usability in varied environments while ensuring the model's utility is maximized across different scenarios.

![offline-inference](../assets/architecture/offline-inference.drawio.png "Offline Inference")

## Training Process

The training cycle begins when an actor (user or system administrator) initiates the creation of a new machine learning model.
This model is then stored in a central database through interactions with the web server.
Following model creation, the actor sets up a training regime that is also registered in the database.

Once initiated, the training enters a loop, repeating for a pre-determined number of iterations.
In each iteration, the clients — distributed computing nodes — download the latest global model from the database.
They then use this global model as a foundation to train local models on their respective datasets.
These local models, containing incremental learning updates, are uploaded back to the web server, which in turn stores them in the database.

After all the local models from the clients have been uploaded, the web server triggers an aggregation task.
This task is responsible for merging the local models to update the global model, utilizing techniques to ensure the global model learns appropriately from the distributed data.
Once the aggregation is complete, the new global model is stored, and any old local models are purged from the database to maintain data hygiene.

Subsequently, the global model undergoes testing to evaluate its performance.
The test metrics generated from this evaluation phase are uploaded to the web server and stored in the database.
The cycle is complete when the pre-determined number of iterations has been completed.
This architecture facilitates continuous improvement of the model while leveraging distributed data sources, optimizing model performance without compromising data privacy or security.

```plantuml
@startuml Training Process
!theme plain
'skinparam responseMessageBelowArrow true
skinparam style strictuml
skinparam sequence {
  LifeLineBorderColor #848484
  GroupBorderColor #848484
}

Actor Actor
participant WebServer
database Database
collections Clients
participant "Celery Task Queue" as celery

activate Actor

Actor -> WebServer ++ : Create Model
WebServer -> Database ++ : Store Model
Database -->> WebServer --
WebServer -->> Actor --

Actor -> WebServer ++ : Create Training
WebServer -> Database ++ : Store Training
Database -->> WebServer --
WebServer -->> Actor --

Actor ->> WebServer --++ : Start Training
WebServer ->> Clients ++ : Start Training
deactivate Clients

loop For n Updates
  WebServer ->> Clients --++ : Start Training Round

  WebServer <- Clients ++ : Download Global Model
  WebServer --> Clients -- : Global Model

  Clients -> Clients ++ : Train Local Model
  Clients -[hidden]-> Clients --

  Clients ->> WebServer --++ : Upload Local Model
  WebServer -> Database ++ : Store Local Model
  Database -->> WebServer --

  note over WebServer,celery #eeeeee
    continue if **all** __//model uploads//__ arrived
  end note

  WebServer ->> celery --++ : Dispatch Aggregation Task

  celery -> Database ++ : Get Local Models
  Database --> celery --

  celery -> celery ++ : Do Aggregation
  celery -[hidden]-> celery --

  celery -> Database ++ : Store "New" Global Model
  Database --> celery --

  celery -> Database ++ : Clean Up Local Models
  Database --> celery --

  celery ->> Clients --++ : Start Model Test

  WebServer <- Clients ++ : Download Global Model
  WebServer --> Clients -- : Global Model

  Clients -> Clients ++ : Test Global Model
  Clients -[hidden]-> Clients --

  Clients ->> WebServer --++ : Upload Global Model Test Metrics
  WebServer -> Database ++ : Store Test Metrics
  Database -->> WebServer --

  note over WebServer,celery #eeeeee
    continue if **all** __//test metrics//__ arrived
  end note
end

WebServer ->> Clients ++ : Training Finished
WebServer ->> Actor --++: Training Finished
deactivate Clients
deactivate Actor
@enduml
```

## Data Model

Our platform contains a polymorphic data model like the depicted below.

```plantuml
@startuml Models
!theme plain

object NotificationReceiver {
  id: UUID
  message_endpoint: URL
}
object User {
  username: Varchar(150)
  first_name: Varchar(150)
  last_name: Varchar(150)
  email: Email
  actor: Boolean
  client: Boolean
}
object Token {
  key: Varchar(40)
  created: DateTime
}
object Model {
  id: UUID
  round: Int
  weights: Binary
}
object GlobalModel {
  name: Varchar(256)
  description: Text
  input_shape: Varchar(100)
}
object SWAGModel {
  swag_first_moment: Binary
  swag_second_moment: Binary
}
object MeanModel
object LocalModel {
  sample_size: Int
}
object Training {
  id: UUID
  state: TrainingState
  target_num_updates: Int
  last_update: DateTime
  aggregation_method: AggregationMethod
  uncertainty_method: UncertaintyMethod
  options: JSON
  locked: Boolean
}
object Metric {
  identifier: Varchar(64)
  key: Varchar(32)
  value: Float | Blob
  step: Int
}

' ===================================
Model "1..*" -up-> "1" User : owner
Model "1" <-left- "*" Metric : model
GlobalModel --|> Model
SWAGModel -up-|> GlobalModel
MeanModel --|> GlobalModel
MeanModel -[hidden]left-> GlobalModel : "\t"
MeanModel "*" --> "1..*" GlobalModel
LocalModel -up-|> Model
LocalModel "1..*" -right-> "1" GlobalModel : base_model
Training "1" --> "1" GlobalModel : model
Training -[hidden]right-> User
Training "1..*" --> "1..*" User : participants
Training "1..*" --> "1" User : actor
Token "1" --> "1" User : user
User -right-|> NotificationReceiver
Metric "1..*" -up-> "1" User : reporter
@enduml
```

```plantuml
@startuml Model Enums
!theme plain
hide enum methods

enum TrainingState {
  I
  O
  C
  E
  S
}
enum AggregationMethod {
  FedAvg
  FedDC
  FedProx
}
enum UncertaintyMethod {
  NONE
  ENSEMBLE
  MC_DROPOUT
  SWAG
}

TrainingState -[hidden]right-> AggregationMethod
AggregationMethod -[hidden]right-> UncertaintyMethod
@enduml
```

## Glossary

See: <https://developers.google.com/machine-learning/glossary?hl=en>
