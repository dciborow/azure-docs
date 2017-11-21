    ---
title: How to create AZTK Batch cluster as compute target for Azure ML
description: Create AZTK Batch cluster as compute targets for Azure ML experimentation. 
services: machine-learning
author: dciborow
ms.author: dciborow
manager: wutao
ms.reviewer: 
ms.service: machine-learning
ms.workload: data-services
ms.topic: article
ms.date: 11/21/2017
---


# Create AZTK Batch cluster as compute target

You can easily scale up or scale out your machine learning experiment by adding additional compute targets such as Apache Spark for AZTK Batch clusters. This article walks you through the steps of creating this compute targets in Azure. For more information on Azure ML compute targets, refer to [overview of Azure Machine Learning experimentation service](experimentation-service-configuration.md).

>[!NOTE]
>You need to ensure you have proper permissions to create resources such as Batch clusters in Azure before you proceed. Also both of these resources can consume many compute cores depending on your configuration. Make sure your subscription has enough capacity for the virtual CPU cores. You can always get in touch with Azure support to increase the maximum number of cores allowed in your subscription.

This process uses the open source toolkit [AZTK](github.com/azure/aztk)

## Prereq 
1. AML Workbench installed and project set up
1. Created an Azure Batch account 

## Steps

1. Clone the AZTK repo
```bash
	git clone -b stable https://www.github.com/azure/aztk

	# You can also clone directly from master to get the latest bits
	git clone https://www.github.com/azure/aztk
```
1. Use pip to install required packages (requires python 3.5+ and pip 9.0.1+), make sure that your pip is for Python3. 
```bash
	pip3 install -r requirements.txt
```
1. Use setuptools:
```bash
	pip3 install -e .
```
1. Move into your AML Workspace Project directory:
```bash
	cd C:\Users\{user}\Documents\AML Workbench\{project}
```
1. Initialize the project in your AML Workspace Project directory [This will automatically create a *.aztk* folder with config files in your working directory]:
```bash
	aztk spark init
```
1. Fill in the fields for your Batch account and Storage account in your *.aztk/secrets.yaml* file. (We'd also recommend that you enter SSH key info in this file)

This package is built on top of two core Azure services, [Azure Batch](https://azure.microsoft.com/en-us/services/batch/) and [Azure Storage](https://azure.microsoft.com/en-us/services/storage/). Create those resources via the portal (see [Getting Started](./docs/00-getting-started.md)).

1. Create and setup your cluster

First, create your cluster:
```bash
aztk spark cluster create \
	--id <my_cluster_id> \
	--size <number_of_nodes> \
	--vm-size <vm_size>
```
You can find more information on VM sizes [here.](https://docs.microsoft.com/en-us/azure/virtual-machines/linux/sizes) Please note that you must use the official SKU name when setting your VM size - they usually come in the form: "standard_d2_v2".

You can also create your cluster with [low-priority](https://docs.microsoft.com/en-us/azure/batch/batch-low-pri-vms) VMs at an 80% discount by using `--size-low-pri` instead of `--size` (we currently do not support mixed low-priority and dedicated VMs):
```
aztk spark cluster create \
	--id <my_cluster_id> \
	--size-low-pri <number_of_low-pri_nodes> \
	--vm-size <vm_size>
```
1. Retrieve SSH information for the cluster.
```bash
aztk spark cluster ssh --id <my_cluster_id> --no-connect
```
1. Attach Compute to AML Workbench
```bash
az ml computetarget attach --name <my_cluster_id> --address <IP>:<Port> --username spark --password <password> --type remotedocker
```
1. Prepare Azure Batch Spark cluster
```bash
az ml experiment prepare -c <my_cluster_id>
```
1. Submit experiments to your cluster like you would any other compute target using either the CLI or Workbench.
```bash
az ml experiment submit -c <my_cluster_id> <mySparkJob.py>
```
