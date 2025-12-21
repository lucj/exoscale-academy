---
type: "page"
id: "ingress"
description: ""
title: "INGRESS"
weight: 6
---

### INGRESS

Route traffic to and from the cluster. Provide a single SSL endpoint for multiple applications. Many implementations of an ingress allow you to customize your platform. Ingresses provide a way to declare that they should channel traffic from the outside of the cluster into destination points within the cluster. One single external Ingress point can accept traffic destined to many internal services.

{{< meshery-design-embed id="embedded-design-ec75661f-1d13-4d63-b643-bf8982a87f8a" src="ingress2.js">}}