---
title: Exercise
weight: 2
---

In this exercise, you'll use a CronJob to create dummy votes

1. In a file named *cronjob.yaml*, define the specification for a *CronJob* resource with the following characteristics:

- Name: seed
- Schedule: "* * * * *"
- Contains a single container with the image *voting/tools:latest* and an environment variable *NUMBER_OF_VOTES* set to 10

2. Deploy the application and verify, from the *resultui* interface, that 10 new votes are created every minute.

3. Delete the application.
