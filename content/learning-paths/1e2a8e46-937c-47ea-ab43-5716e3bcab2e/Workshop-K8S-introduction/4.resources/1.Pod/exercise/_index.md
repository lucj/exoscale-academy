---
title: Exercise
weight: 2
---

In this exercise, you will run the VotingApp components in Pods.

1. Create the *votingapp* folder on your local machine. Inside this one, create the YAML files containing the Pod specifications for each microservice, following the instructions from the table below:

    | Microservice | File Name          | Pod Name   | Container Image         |
    | ---          | ---                | ---        | ---                     |
    | Vote UI      | pod-voteui.yaml    | vote-ui    | voting/vote-ui:latest   |
    | Vote         | pod-vote.yaml      | vote       | voting/vote:latest      |
    | Redis        | pod-redis.yaml     | redis      | redis:7.0.8-alpine3.17  |
    | Worker       | pod-worker.yaml    | worker     | voting/worker:latest    |
    | Postgres     | pod-db.yaml        | db         | postgres:15.1-alpine3.17|
    | Result       | pod-result.yaml    | result     | voting/result:latest    |
    | Result UI    | pod-resultui.yaml  | result-ui  | voting/result-ui:latest |

    For the *db* Pod, make sure to specify an environment variable *POSTGRES_PASSWORD* with the value *postgres*.

2. Deploy the application defined by all of these specifications

3. What do you notice ?

4. Delete the application
