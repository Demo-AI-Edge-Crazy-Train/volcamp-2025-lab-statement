+++
title = "Start services"
draft= false
weight= 8
+++


1. From Devspaces, click on the search bar on the top and choose "Run Task" from the drop-down list.
! [Run Task menu](/images/dev-section/select-run-task.png)

2. Select the **start-all-apps** task, this task will run all the previously modified applications in parallel. 

![ Start all applications](/images/dev-section/start-all-aps-task.png)

3. Click on **Continue without scanning the task output**.

![ Scan task](/images/dev-section/scan-task-output.png)

4. Each application will start in a terminal. Terminals are accessible from the bottom right, 

Select 'no' on the pop-up which indicates that a new process has started and that it is possible to make a port redirection.

 ![Start tasks](/images/dev-section/all-tasks-started.png)

5. To check that all the applications have started, you should have the following logs:
- **Log capture-app**
![capture-app application log](/images/dev-section/start-capture-log.png)

-  **intelligent-train log**
![Log intelligent-train application](/images/dev-section/intelligent-train-log.png)

- **Log train-ceq-app**
![Train-ceq-app log](/images/dev-section/train-ceq-log.png)

-  Monitoring-app log**! 
![Monitoring-app application log](/images/dev-section/train-monitoring-log.png)

- **Log train-controller** 
![Train-controller application log](/images/dev-section/train-controller-log.png)


Now that all the applications have been started, we're going to simulate the operation of our intelligent train!  
