---
title: Using Git Repo with an Azure Machine Learning Workbench Project | Microsoft Docs
description: This article explain how to use a Git repository in conjunction with an Azure Machine Learning Workbench Project. 
services: machine-learning
author: hning86
ms.author: haining
manager: haining
ms.reviewer: garyericson, jasonwhowell, mldocs
ms.service: machine-learning
ms.workload: data-services
ms.topic: article
ms.date: 11/16/2017
---
# Using Git repository with an Azure Machine Learning Workbench project
This document provides information on how Azure Machine Learning Workbench uses Git to ensure reproducibility in your data science experiment. Instructions on how to associate your project with a cloud Git repository are also provided.

## Introduction
Azure Machine Learning Workbench is designed with Git integration from the ground up. When creating a new project, the project folder is automatically "Git-initialized" as a local Git repository (repo) while a second hidden local Git repo is also created with a branch named _AzureMLHistory/<project_GUID>_ to keep track of project folder changes for each execution. 

Associating the Azure ML project with a Git repo, hosted within a Visual Studio Team Service (VSTS) project, enables automatic version-control both locally and remotely. This association lets anybody with access to the remote repo download the latest source code to another computer (roaming).  

> [!NOTE]
> VSTS has its own access control list that is independent of Azure Machine Learning Experimentation Service. User access may vary between a Git repo and an Azure ML workspace or project, and may need to be managed. So if you want to share your Azure ML Project with a team member including code level access, in addition to just share the workspace, you need to explicitly grant him/her proper access to the VSTS Git repo. 

With Git, it is also possible to manage version control explicitly by using the _master_ branch or by creating other branches on the repo. You can just use the local Git repo, and you can also push to the remote Git repo if provisioned.

This diagram depicts the relationship between a VSTS Git repo and an Azure ML project:

![AML Git](media/using-git-ml-project/aml_git.png)

To get started using a remote Git repo, follow these basic instructions.

> [!NOTE]
> Currently, Azure Machine Learning only supports Git repositories on VSTS Accounts. Support for General Git repos (such as GitHub and etc.) is planned in the future.

## Step 1. Create an Azure ML Experimentation Account
If not already done, create an Azure ML Experimentation Account and install the Azure ML Workbench app. See more details in the [Install and create Quickstart](quickstart-installation.md).

## Step 2. Create a Team project or use an existing Team project
From [Azure portal](https://portal.azure.com/), create a new **Team Project**.
1. Click on **+**
2. Search for **"Team Project"**
3. Enter the required information.
    - Name: A team name.
    - Version Control: **Git**
    - Subscription: The one with an Azure Machine Learning Experimentation Account.
    - Location: Ideally stay in a region that is close to your Azure Machine Learning Experiment resources.
4. Click **Create**. 

![Create a Team Project from Azure Portal](media/using-git-ml-project/create_vsts_team.png)


> [!TIP]
> Make sure to sign-in with the Azure Active Directory (AAD) account used to access the Azure Machine Learning Workbench. Otherwise, the new team project might end-up under the wrong Tenant-ID and Azure Machine Learning might not find it. In this case, you would have to use the command-line interface and supply the VSTS token.

Once the Team Project is created, you are ready to move to the next step.

To navigate directly to the Team Project just created, the URL is `https://<team_project_name>.visualstudio.com`.

> [!NOTE]
> Currently, Azure Machine Learning only supports empty Git repos with no master branch. From the command-line interface, you can use the --force argument to delete your master branch first. 

## Step 3. Create a new Azure ML project with a remote Git repo
Launch Azure ML Workbench and create a new project. Fill the Git repo text box with the VSTS Git repo URL that you get from Step 2. It typically looks like this: `http://<vsts_account_name>.visualstudio.com/_git/<project_name>`

![Create Azure ML Project with Git repo](media/using-git-ml-project/create_project_with_git_rep.png)

Now a new Azure ML project is created with remote Git repo integration enabled and ready to go. The project folder is always Git-initialized as a local Git repo. And the Git _remote_ is set to the remote VSTS Git repo so commits can be pushed into the remote Git repo.

## Step 3a. Associate an existing Azure ML project with a VSTS Git repo
Optionally, you can also create a Azure ML project without a VSTS Git repo, and just rely on the local Git repo for run history snapshots. And you can associate a VSTS Git repo later with this existing Azure ML project using the following command:

```azurecli
# make sure you are in the project path so CLI has context of your current project
$ az ml project update --repo http://<vsts_account_name>.visualstudio.com/_git/<project_name>
```

## Step 4. Capture project snapshot in Git repo
Now you can execute a few runs in the project, make some changes in-between the runs. You can do this either from the desktop app, or from CLI using `az ml experiment submit` command. For more details, you can follow the [Classifying Iris tutorial](tutorial-classifying-iris-part-1.md). For each run, if there is any change made in any files in the project folder, a snapshot of the entire project folder is committed and pushed into the remote Git repo under a branch named `AzureMLHistory/<Project_GUID>`. You can view the branches and commits by browsing to the VSTS Git repo URL and find this branch. 

![run history branch](media/using-git-ml-project/run_history_branch.png)

Note it is better not operate in the history branch yourself. Doing so might mess up with run history. Use master branch or create other branches instead for your own Git operations.

## Step 5. Restore a previous project snapshot 
To restore the entire project folder to the state of a previous run history project state snapshot, from Azure ML Workbench:
1. Click on **Runs** in the activity bar (glass-hour icon).
2. From the **Run List** view, click on the run you want to restore.
3. From the **Run Detail** view, click on **Restore**.

![run history branch](media/using-git-ml-project/restore_project.png)

Alternatively, you can use the following command from the Azure ML Workbench CLI window.

```azurecli
# discover the run I want to restore snapshot from:
$ az ml history list -o table

# restore the snapshot from a particular run
$ az ml project restore --run-id <run_id>
```

By executing this command, we will overwrite the entire project folder with the snapshot taken when that particular run was kicked off. But your project stays on the current branch. This means that you will **lose all changes** in your current project folder. So please be extra careful when you run this command.

## Step 6. Use the master branch
One way to avoid accidentally losing your current project state, is to commit the project to the master branch (or any branch you created yourself) of the Git repo. You can directly use Git from command line (or your other favorite Git client tool of choice) to operate on the master branch. For example:

```
# make sure you are on the master branch (or branch of your choice)
$ git checkout master

# stage all changes
$ git add -A

# commit all changes locally on the master branch
$ git commit -m 'this is my updates so far'

# push changes into the remote VSTS Git repo master branch.
$ git push origin master
```

Now you can safely restore project to an earlier snapshot following Step 5, knowing that you can always come back to the commit you just made on the master branch.

## Authentication
If you just rely the run history functions in Azure ML for taking project snapshots and restoring them, you don't need to worry about Git repo authentication. It is taken care by the Experimentation Service layer.

However, if you use your own Git tools to manage version control, you will need to properly handle authentication against the remote Git repo on VSTS. That is, you will need to set up authentication with the Git repo on the local computer before you can issue Git commands against that remote Git repo. 

The easiest way to do this, is to create an SSH key pair and upload the public key portion to the Git repo security settings.

### Generate SSH key 
First let's generate a pair of SSH keys on your computer.

#### On Windows:
Launch Git GUI desktop app on Windows, and under _Help_ menu, click on _Show SSH Key_.

![SSH Key](media/using-git-ml-project/git_gui.png)

Copy SSH into clipboard.

#### On macOS:
Quick steps from command shell:
```
# generate the SSH key
$ ssh-keygen -t rsa -b 4096 -C "your_email@example.com"

# start the SSH agent in the background
$ eval "$(ssh-agent -s)"

# add newly generated SSH key to the SSH agent
$ ssh-add -K ~/.ssh/id_rsa

# display the public key so you can copy it.
$ more ~/.ssh/id_rsa.pub
```
More details steps can be found on [this GitHub article](https://help.github.com/articles/generating-a-new-ssh-key-and-adding-it-to-the-ssh-agent/).

### Upload public key to Git repo
Go to your Visual Studio account homepage: https://<vsts_account_name>.visualstudio.com and log in, then click on Security under your avatar.

Then add a SSH public key,  paste the SSH public key you get from the previous step, and give it a name. You can add multiple keys here.

Now you can issue Git commands locally against the remove repo with no further explicit authentication required.

### Read more
Please follow these two articles (either approach can work) for more details on how to enable local authentication to the remote Git repo in VSTS.
- [Use SSH Key Authentication](https://www.visualstudio.com/en-us/docs/git/use-ssh-keys-to-authenticate)
- [Use Git Credential Managers](https://www.visualstudio.com/en-us/docs/git/set-up-credential-managers)


## Next steps
Learn how to use the Team Data Science Process to organize your project structure, see [Structure a project with TDSP](how-to-use-tdsp-in-azure-ml.md)
