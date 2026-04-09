---
title: Exercise
weight: 2
---

In this exercise, you'll use a Secret to secure the connection to Postgres

1. In a *secret-db.yaml* file, add the specification for a Secret containing the key *password* with the associated value *dbpass*.

2. Modify the *db* Deployment to reference this Secret key (instead of specifying the password in plain text).

3. Add the POSTGRES_PASSWORD environment variable in the containers of the *worker* and *result* Deployments, ensuring that the value of this variable references the key of the Secret created earlier.

4. Deploy the application defined in this specification and verify that you have access to both the voting and result interfaces.

5. Delete the application.
