---
title: Creating an SKS cluster
weight: 1
---

SKS (Scalable Kubernetes Service) is the Exoscale-managed cluster. To deploy an SKS cluster, we can use one of the following tools:

- [Terraform](https://terraform.io)
- [Pulumi](https://pulumi.com)
- [exo cli](https://github.com/exoscale/cli)
- [Exoscale Portal](https://exoscale.com)

Whatever tool we use, it creates the following resources:  

- a Security Group and some associated rules
- an SKS cluster
- a Nodepool

We will go through the details of each approach, so you can select the one you prefer. But first, you need to create an account and get free credits to follow this workshop. 
