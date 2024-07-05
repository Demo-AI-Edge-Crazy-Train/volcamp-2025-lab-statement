+++
title = "Start services"
draft= false
weight= 8
+++


1. A partir de Devspaces, cliquez sur la barre de recherche en haut et choisir dans la liste déroulante "Run Task"
 ![Menu Run Task](/images/dev-section/select-run-task.png)

2. Sélectionnez la tâche **start-all-apps**, cette tâche lance en parallèle toutes les applications modifiées précédement. 

 ![Démarrer toutes les applications](/images/dev-section/start-all-aps-task.png)

3. Cliquez sur **Continue without scanning the task outoput**

 ![Scan de tâche](/images/dev-section/scan-task-output.png)

4. Chaque application démarrera dans un terminal. Les terminaux sont accessible en bas à droite, 

Séléctionnez 'no' sur les pop-up qui indique qu'un nouveau processus a démarré et qu'il est possible de faire une redirection de port.

 ![Démarrage des tâches](/images/dev-section/all-tasks-started.png)

5. Pour vérifier que toutes les application ont bien démarrés, vous devriez avoir les logs suivants :
- **Log capture-app**
![Log de l'application capture-app](/images/dev-section/start-capture-log.png)

- **Log intelligent-train**
![Log de l'application intelligent-train](/images/dev-section/intelligent-train-log.png)

- **Log train-ceq-app**
![Log de l'application train-ceq-app](/images/dev-section/train-ceq-log.png)

- **Log monitoring-app** 
![Log de l'application monitoring-app](/images/dev-section/train-monitoring-log.png)

- **Log train-controller** 
![Log de l'application train-controller](/images/dev-section/train-controller-log.png)


Maintenant que toutes les applications sont démarrées, nous allons simuler le fonctionnement de notre train intelligent !  
