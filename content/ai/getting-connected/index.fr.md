+++
title = "Connection"
draft= false
weight= 1
[[resources]]
  src = '**.png'
+++

Pour les besoins de ce lab, nous avons provisionné un seul cluster OpenShift, avec OpenShift AI déployé dessus.

Chaque participant dispose d'un compte utilisateur unique pour effectuer son travail.

## Informations sur l'environnement

Ouvrez l'URL suivante et connectez-vous :

* L'URL Red Hat OpenShift AI Dashboard pour notre environnement partagé : [https://rhods-dashboard-redhat-ods-applications.apps.{{< param openshift_domain >}}](https://rhods-dashboard-redhat-ods-applications.apps.{{< param openshift_domain >}})
* Entrez vos informations d'identification (distribuées sur un papier pendant le lab).
* Le résultat devrait ressembler à :

![02-01-login1](02-01-login1.png)


* Parce que le mot de passe est simple, votre navigateur peut afficher un message tel que :
![02-01-login-scary](02-01-login-scary.png)
* Vous pouvez ignorer ce message lorsqu'il apparaît.

* Après l'authentification, le résultat devrait ressembler à ce qui suit :
![02-01-rhoai-front-page](02-01-rhoai-front-page.png)

Si vous êtes arrivé jusqu'ici et que vous avez vu tout cela, félicitations, vous vous êtes correctement connecté à l'application OpenShift AI Dashboard !

Nous sommes maintenant prêts à commencer le lab.