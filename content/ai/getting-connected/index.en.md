+++
title = "Getting Connected"
draft= false
weight= 1
[[resources]]
  src = '**.png'
+++

For the purposes of this training session, we have provisioned a single OpenShift cluster, with OpenShift AI deployed on it.

Each person attending this lab will have a unique user account in which to do their work.

## Environment information

In a new window or tab, open the following URL and log in:

[hugo](https://gohugo.io/)

* The Red Hat OpenShift AI Dashboard URL for our shared environment: [https://rhods-dashboard-redhat-ods-applications.apps.{{< param openshift_domain >}}](https://rhods-dashboard-redhat-ods-applications.apps.{{< param openshift_domain >}})
* Enter your credentials (distributed on a paper during the lab)
* The result should look like:

![02-01-login1](02-01-login1.png)



* Because the password is so simple, your browser might display a scary message such as:
![02-01-login-scary](02-01-login-scary.png)
* It is safe here to ignore this message when it pops up.

* After you authenticate, the result should look like:
![02-01-rhoai-front-page](02-01-rhoai-front-page.png)

If you got this far and saw all that, congratulations, you properly connected to the OpenShift AI Dashboard Application!

We are now ready to start the lab.