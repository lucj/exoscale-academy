---
title: Exercise
weight: 2
---

In this exercise, you'll use Deployments in the VotingApp

1. In the *votingapp* directory, replace each Pod specification with a Deployment specification with a single replica. Name these Deployment files *deploy-XXX.yaml* where XXX is the name of the microservice (*voteui*, *vote*, ...)

2. Deploy the application defined by these specifications

3. Access the vote and result interfaces via NodePort Services

4. Delete a Pod. What happens ?

5. Delete the application
