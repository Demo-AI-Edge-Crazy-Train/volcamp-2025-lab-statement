+++
title = "Test services"
draft = false
weight = 9
+++

We are going to simulate the operation of the train:
1. Open a new terminal

![terminal](/images/dev-section/new-terminal-bash.png)

2. Execute the command below to retrieve the URL of the monitoring console:  
```
oc get routes -o jsonpath='{range .items[*]}{.metadata.annotations.che\.routing\.controller\.devfile\.io/endpoint-name}{"\t"}http://{.spec.host}{"\n"}{end}' | grep monitoring-svc | cut -f 2
```
![URL monitoring](/images/dev-section/get-url-monitoring.png)

3. Copy the URL, launch a new browser window in anonymous mode (in order to have an empty cache), insert the URL.

![URL monitoring](/images/dev-section/monitoring-console.png)

{{% notice warning %}}
**Google Chrome** seems to transform the HTTP URL into an HTTP**S** one.
If you get an error message saying *"Application is not available. The application is currently not serving requests at this endpoint. It may not have been started or is still starting."*, please double check the URL scheme (HTTP/HTTPS) or open the URL with another Browser.
Firefox seems to work just fine for this usage.
{{% /notice %}}

4. Return to your terminal and run the following command:
```
curl -X 'POST' 'http://localhost:8082/capture/test' -H 'accept: */*'
```


5. From your browser, you should be able to see the train simulation and the traffic sign detection in real time. If not, refresh your browser.

![streaming video](/images/dev-section/streaming-video.png)

Well done! The simulation worked well :) now you can stop the simulation by closing all the terminals.