---
title: Exercise
weight: 2
---

In this exercise, you'll add Services to the VotingApp

1. In the *votingapp* directory, create YAML files containing the specifications for the Services of each microservice in the application, according to the table below:

    | Microservice | File Name         | Service Type     | Service Details                          |
    | ---          | ---               | ---              | ---                                      |
    | Vote UI      | svc-voteui.yaml   | NodePort (31000) | nodePort 31000, port: 80, targetPort: 80 |
    | Vote         | svc-vote.yaml     | ClusterIP        | port: 5000, targetPort: 5000             |
    | Redis        | svc-redis.yaml    | ClusterIP        | port: 6379, targetPort: 6379             |
    | Postgres     | svc-db.yaml       | ClusterIP        | port: 5432, targetPort: 5432             |
    | Result       | svc-result.yaml   | ClusterIP        | port: 5000, targetPort: 5000             |
    | Result UI    | svc-resultui.yaml | NodePort (31001) | nodePort 31001, port: 80, targetPort: 80 |

    Note that it is not necessary to expose the *worker* Pod with a Service as no Pod needs to connect to it. Instead, it is the *worker* Pod that connects to *redis* and *db*.

    For each Pod/Service pair, make sure to properly define a label in the Pod and the corresponding *selector* in the Service.

2. Deploy the application defined by these specifications

3. Access the vote and result interfaces via the NodePort Services

4. Delete the application
