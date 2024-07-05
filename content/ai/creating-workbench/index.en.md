+++
title = "Creating a workbench"
draft= false
weight= 3
[[resources]]
  src = '**.png'
+++

## Launch a Workbench

* Once the Data Connection and Pipeline Server are fully created
* Create a workbench
![02-03-create-wb.png](02-03-create-wb.png)
* Make sure it has the following characteristics:  
    * Choose a name for it, like: `My Workbench`  
    * Image Selection: `CUSTOM Crazy train lab`
    * Container Size: `Small`
    * Keep the default cluster storage settings
    * On the bottom, tick "**Use a data connection**"
    * Scroll down to "**Use existing data connection**"
    * Select from the list the "**pipelines**" data connection you previously created.
    * That should look like:
![02-02-launch-workbench-01.png](02-02-launch-workbench-01.png)
![02-02-launch-workbench-02.png](02-02-launch-workbench-02.png)
* Create the workbench and wait for your workbench status to be “Running” 
* Once it is, click the **Open** Link to connect to it.
![02-03-open-link.png](02-03-open-link.png)

* Authenticate with the same credentials as earlier
* You will be asked to accept the following settings:
![02-02-accept.png](02-02-accept.png)

* Do so
* You should now see this:
![02-02-jupyter.png](02-02-jupyter.png)

## Git-Clone the lab repo

We will clone the content of our Git repo so that you can access all the materials that were created as part of our prototyping exercise.

* Using the Git UI:
  * Open the Git UI in Jupyter:
![git-clone-1.png](git-clone-1.png)
* Enter the URL of the Git repo: ```{{< param gitAIRepoUrl >}}```. Select also "Download the repository".
![git-clone-2.png](git-clone-2.png)

At this point, your project is ready for the work we want to do in it.
