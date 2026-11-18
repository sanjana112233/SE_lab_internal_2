# Kubernetes and Nagios – Lab Commands

**Run commands in Windows PowerShell or VS Code Terminal**, except commands marked for the browser or Kubernetes YAML files.

---

# PART A – KUBERNETES (SET-7)

## 1. Check prerequisites

Start Docker Desktop first.

Open PowerShell / VS Code Terminal:

```powershell
docker --version
minikube version
kubectl version --client
```

If Minikube or kubectl is not installed, install them before continuing. Minikube installer (PowerShell):

```powershell
curl.exe -LO https://storage.googleapis.com/minikube/releases/latest/minikube-installer.exe
.\minikube-installer.exe
```

Close and reopen the terminal after installation.

## 2. Start Minikube

Use Docker Desktop as the driver:

```powershell
minikube start --driver=docker
minikube status
kubectl get nodes
```

If the cluster is already running, continue to Step 3.

## 3. Get the Metro Reservation project

Run these commands in the folder where you want the project:

```powershell
git clone https://github.com/deepthisagar7/metro-reservation.git
cd metro-reservation
mvn clean package
dir target
```

Check the exact WAR filename inside `target`.

## 4. Create the application Dockerfile

In the `metro-reservation` project root, create a file named `Dockerfile`.

Paste this (change the WAR filename if `dir target` shows a different name):

```dockerfile
FROM tomcat:9
RUN rm -rf /usr/local/tomcat/webapps/*
COPY target/metro-reservation.war /usr/local/tomcat/webapps/ROOT.war
EXPOSE 8080
CMD ["catalina.sh", "run"]
```

## 5. Build the image for Minikube

Run in the same project folder:

```powershell
minikube image build -t metro-reservation:latest .
```

## 6. Create Kubernetes Deployment

Create a file named `metro-deployment.yaml` in the project folder.

Paste:

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: metro-reservation
spec:
  replicas: 2
  selector:
    matchLabels:
      app: metro-reservation
  template:
    metadata:
      labels:
        app: metro-reservation
    spec:
      containers:
        - name: metro-reservation
          image: metro-reservation:latest
          imagePullPolicy: IfNotPresent
          ports:
            - containerPort: 8080
```

Apply it:

```powershell
kubectl apply -f metro-deployment.yaml
kubectl get deployments
kubectl get pods
kubectl describe deployment metro-reservation
```

## 7. Expose the application

Create a file named `metro-service.yaml`.

Paste:

```yaml
apiVersion: v1
kind: Service
metadata:
  name: metro-service
spec:
  type: NodePort
  selector:
    app: metro-reservation
  ports:
    - port: 8080
      targetPort: 8080
      nodePort: 30080
```

Apply and check:

```powershell
kubectl apply -f metro-service.yaml
kubectl get services
```

Open the application through port forwarding. Keep this terminal running:

```powershell
kubectl port-forward service/metro-service 8081:8080
```

Browser: `http://localhost:8081`

## 8. Scale up and scale down

Scale up to 4 pods:

```powershell
kubectl scale deployment metro-reservation --replicas=4
kubectl get pods
kubectl get deployments
```

Scale down to 2 pods:

```powershell
kubectl scale deployment metro-reservation --replicas=2
kubectl get pods
```

## 9. Demonstrate automatic pod recreation

List pods:

```powershell
kubectl get pods
```

Delete one pod (replace POD_NAME with an actual pod name from the output):

```powershell
kubectl delete pod POD_NAME
kubectl get pods
```

Kubernetes creates a replacement pod to maintain the desired replica count.

## 10. Kubernetes troubleshooting commands

```powershell
kubectl get pods
kubectl describe pod POD_NAME
kubectl logs POD_NAME
kubectl logs POD_NAME --previous
kubectl get events
kubectl get deployments
kubectl rollout status deployment/metro-reservation
kubectl rollout history deployment/metro-reservation
kubectl rollout undo deployment/metro-reservation
kubectl get services
kubectl describe service metro-service
```

## 11. Dashboard and cluster commands

```powershell
minikube dashboard
minikube status
minikube stop
minikube start
```

## 12. Clean up Kubernetes resources

```powershell
kubectl delete -f metro-service.yaml
kubectl delete -f metro-deployment.yaml
minikube stop
```

Optional: delete the Minikube cluster only if you no longer need it:

```powershell
minikube delete
```

---

# PART B – NAGIOS (SET-7)

## 1. Pull the Nagios image

Open PowerShell / VS Code Terminal. Docker Desktop must be running.

```powershell
docker pull jasonrivers/nagios:latest
```

## 2. Run Nagios

```powershell
docker run -d --name nagiosdemo -p 8888:80 --restart unless-stopped jasonrivers/nagios:latest
```

## 3. Check the container

```powershell
docker ps
docker logs nagiosdemo
```

## 4. Open Nagios dashboard

Open browser:

`http://localhost:8888`

Login:

```text
Username: nagiosadmin
Password: nagios
```

In the dashboard, open **Hosts**, **Services** and **Alerts** to view monitoring status.

## 5. Check Nagios container status and logs

```powershell
docker ps
docker logs nagiosdemo
docker logs -f nagiosdemo
```

Press `Ctrl + C` to stop viewing live logs. This does not stop the container.

## 6. Restart / stop / start Nagios

```powershell
docker restart nagiosdemo
docker stop nagiosdemo
docker start nagiosdemo
```

## 7. Remove Nagios

```powershell
docker stop nagiosdemo
docker rm nagiosdemo
```

Optional image removal:

```powershell
docker images
docker rmi jasonrivers/nagios:latest
```

---

# PART C – SET-7 SCENARIO QUESTIONS

## Kubernetes

**1. One application pod crashes during ticket booking.**

```powershell
kubectl get pods
kubectl describe pod POD_NAME
kubectl logs POD_NAME
```

Kubernetes detects the failed pod and the Deployment creates a replacement to maintain the desired replica count.

**2. How does Kubernetes maintain replicas?**

The Deployment controller continuously compares actual pods with the desired replica count and creates or removes pods as needed.

```powershell
kubectl get deployments
kubectl scale deployment metro-reservation --replicas=4
```

**3. Scaling up vs scaling down**

```powershell
kubectl scale deployment metro-reservation --replicas=4
kubectl scale deployment metro-reservation --replicas=2
```

Scaling up increases pods. Scaling down decreases pods.

**4. How do more replicas help during peak hours?**

More application pods can handle requests in parallel, improving capacity and availability, subject to available resources.

**5. How are requests distributed among pods?**

A Kubernetes Service selects matching pods and distributes network traffic among the available endpoints.

```powershell
kubectl get services
kubectl describe service metro-service
```

## Nagios

**1. How does Nagios identify whether the application server is up or down?**

Nagios checks host availability and displays UP or DOWN status in the dashboard.

**2. WARNING vs CRITICAL**

WARNING means a monitored value has crossed a warning threshold. CRITICAL means a serious threshold or service failure has occurred.

**3. What if CPU usage reaches 100%?**

Nagios can show an alert if CPU monitoring is configured. High CPU can slow down or affect the application.

**4. How can Nagios detect an unavailable application when the server is running?**

Configure an HTTP service check for the application's URL. A failed HTTP check can report the application as unavailable even if the host is UP.

**5. How can the administrator identify whether downtime is caused by Tomcat, MySQL or the server?**

Monitor each separately: host availability, Tomcat HTTP/service check and MySQL service/port check. Compare their statuses and logs.

## Additional Kubernetes practical questions from the lab notes

**Pod keeps restarting:**

```powershell
kubectl get pods
kubectl describe pod POD_NAME
kubectl logs POD_NAME
kubectl logs POD_NAME --previous
```

**Pod stuck in Pending:**

```powershell
kubectl describe pod POD_NAME
kubectl get events
kubectl get nodes
```

Check events for insufficient CPU/memory, scheduling problems or image-pull issues.

**Debug a failed Deployment:**

```powershell
kubectl get deployments
kubectl describe deployment metro-reservation
kubectl get pods
kubectl describe pod POD_NAME
kubectl logs POD_NAME
kubectl get events
```

**Pods failing health checks:**

```powershell
kubectl describe pod POD_NAME
kubectl logs POD_NAME
kubectl get events
```

Check the application logs, probe settings, ports and resource availability.

**Roll back a faulty Deployment:**

```powershell
kubectl rollout history deployment/metro-reservation
kubectl rollout undo deployment/metro-reservation
kubectl rollout status deployment/metro-reservation
```

**Debug a running pod:**

```powershell
kubectl exec -it POD_NAME -- sh
```

Exit the pod shell:

```sh
exit
```

**Expose a local service:**

```powershell
kubectl port-forward service/metro-service 8081:8080
```

Open `http://localhost:8081`.

## Additional Nagios practical questions from the lab notes

**Start and stop Nagios:**

```powershell
docker start nagiosdemo
docker stop nagiosdemo
```

**Nagios web interface says “Unable to connect to Nagios process”:**

```powershell
docker ps -a
docker logs nagiosdemo
docker restart nagiosdemo
```

If it continues, inspect Nagios service/process and configuration inside the container.

**New host not appearing:**

Check that the host configuration exists, has valid syntax, is included in the Nagios configuration, and Nagios has reloaded/restarted after the change. The supplied lab image does not provide a complete custom-host configuration procedure.

**Check whether Nagios is running:**

```powershell
docker ps
docker logs nagiosdemo
```

**View Nagios logs live:**

```powershell
docker logs -f nagiosdemo
```

**Advantages of Nagios:**

- Monitors servers, networks, applications and services.
- Detects failures and performance issues.
- Displays status and alerts.
- Helps administrators identify and troubleshoot downtime.

---

# IMPORTANT NOTES

1. Run Kubernetes commands in PowerShell / VS Code Terminal on your computer, not inside a Docker container.
2. Keep Docker Desktop running before starting Minikube or Nagios.
3. Keep the `kubectl port-forward` terminal open while testing the browser.
4. The Metro app must be compatible with Tomcat 9 and the WAR filename in the Dockerfile must match the actual file in `target`.
5. Kubernetes replicas run the application image. The database/JDBC configuration is a separate requirement; scaling app pods alone does not configure or replicate MySQL.
6. The Nagios image starts the dashboard. Monitoring the actual Metro server's CPU, memory, disk, Tomcat and MySQL requires configuring suitable host/service checks and agents/plugins; opening the dashboard alone does not automatically configure all those checks.
7. If Minikube reports an old profile/configuration or Windows access-denied error, do not delete Minikube folders manually. Check `minikube status` and `minikube logs` first.

