# k8s-repo  spiekbriefje

## Kubernetes objecten maken

Om de applicatie straks op het Kubernetes platform te laten draaien, moeten een aantal Kubernetes objecten gemaakt worden. Een Kubernetes object kan aangemaakt worden door een beschrijving van dat object in een tekstbestand op te nemen, ook wel een [.yaml file](https://kubernetes.io/docs/concepts/overview/working-with-objects/kubernetes-objects/) genoemd - en deze straks in de GitLab CI/CD pipeline daadwerkelijk toe te passen. Omdat we in deze tutorial niet een hele Kubernetes cursus kunnen geven, adviseren wij je om eens een kijkje te nemen op https://www.kubernetes.io. Voor deze tutorial is het echter voldoende dat je de volgende drie objecten kent:

 - Pod: de laagste abstractie-laag om een containers heen. Meestal heeft een pod één container, maar het is ook mogelijk om meerdere containers aan een pod toe te voegen. De functie van de pod is eigenlijk puur het doorsturen van acties voor de container. Omdat pods worden gedefinieerd middels een deployment, hoeven we niet een aparte yaml file te maken voor een pod.
 - Deployment: de middelste abstractie-laag in de container-virtualisatie van Kubernetes. Deployments regelen alles met betrekking tot scaling en monitoring van pods. Als een pod faalt, zorgt de deployment ervoor dat er een nieuwe wordt opgespind. Up- en down-scaling gebeurt ook hier. Deployments kunnen verschillende soorten pods hebben, maar over het algemeen is het beter om dit per deployment bij één type te houden. Dit komt de beheersbaarheid en overzichtelijkheid ten goede.
 - Service: de hoogste abstractie-laag van de container-virtualisatie. Services kunnen worden gebruikt om communicatie te faciliteren tussen deployments, en om communicatie van buitenaf mogelijk te maken (port forwarding bijvoorbeeld) via een Ingress (zie hierna). Wanneer er vanuit een pod gecommuniceerd wordt naar een andere container, gaat dit dus altijd via een service.
 - Ingress: Ingress is een methode om http– en https-services te kunnen bereiken vanaf een externe locatie, van buiten het Kubernetes cluster. Binnen de Logius Private Cloud hebben wij een zogenaamde "Ingress Controller" geïnstalleerd, die het cluster in staat stelt om Ingress objecten die door jou worden aangemaakt te herkennen. Specifiek maken wij gebruik van de Nginx Ingress Controller.
 
## Vm setup: Kubernetes Linux DisciplVM 5.0.0-1027-azure #29~18.04.1-Ubuntu 
          SMP Mon Nov 25 21:18:57 UTC 2019 x86_64 x86_64 x86_64 GNU/Linux

IP:     discipl.westeurope.cloudapp.azure.com (51.136.24.239) 
User:   boscp08
Password: Discipl_D
url  :   


````
Welcome to Ubuntu 18.04.3 LTS (GNU/Linux 5.0.0-1027-azure x86_64)

 * Documentation:  https://help.ubuntu.com
 * Management:     https://landscape.canonical.com
 * Support:        https://ubuntu.com/advantage

  System information as of Sun Dec 15 18:05:34 UTC 2019

  System load:  0.0                Processes:              113
  Usage of /:   13.9% of 28.90GB   Users logged in:        1
  Memory usage: 25%                IP address for eth0:    10.0.0.4
  Swap usage:   0%                 IP address for docker0: 172.17.0.1

 * Overheard at KubeCon: "microk8s.status just blew my mind".

     https://microk8s.io/docs/commands#microk8s.status

 * Canonical Livepatch is available for installation.
   - Reduce system reboots and improve kernel security. Activate at:
     https://ubuntu.com/livepatch

0 packages can be updated.
0 updates are security updates.


Last login: Sun Dec 15 17:30:47 2019 from 86.86.102.241


````
 # Sheets 
 
 ## Kubectl, onze beste vriend (veel voorkomende commando's)
 
 `kubectl [command] [TYPE] [NAME] [flags] `

`get [pod|service|deployment|and more...] `

`describe [pod|service|deployment|and more...] `

`exec -ti podname – command `

`logs podname (-f for keeping log open) `

`create –f filename `

`delete –f filename  `
 
 volledige overzicht blader naar  https://kubernetes.io/docs/reference/kubectl/overview/
 
 ## Laten we een paar veelgebruikte kubectl-opdrachten uitvoeren
 
`kubctl get namespaces `

`kubectl get pod –-all-namespaces –o wide `

`kubectl get pod –n kube-system `

`kubectl get service --all-namespaces –o wide `

`kubectl get service -n kube-system `

`kubectl describe pod -n kube-system coredns-... `

`kubectl describe service -n kube-system coredns `

Beetje geavanceerder ..

`kubectl describe pod -n kube-system $(kubectl get pod -n kube-system -o name| cut -d/ -f2- |grep dns)`


 dashboard under contruction

https://discipl.westeurope.cloudapp.azure.com 
 
## mysql probeersel 
````
wijzig  the mysql configfile 
Browse to ~/masterclass/mysql 
Generate the hash for your password (choose your own password) 
`echo -n "ThisIsCool" |base64 `
Save the output  (remember your password) 
Open mysql.yaml with an editor and modify: 

data:
  password: `insert your generated base64 hash`

Save the file!  


Start mysql in Kubernetes 

kubectl create -f mysql.yaml kubectl get pod -n boscp08  (where -n is the name of your namespace)
kubectl get service -n student12 
Create your database (Modify the password if you picked your own): 
cat start.sql| kubectl -n student12 exec $(kubectl get pod -n student12 -o name | cut -d/ -f2- | grep mysql) -ti -- mysql -u root --password=ThisIsCool 

If you got the output “unable to use a TTY” you have been successful. kubectl -n student12 logs $(kubectl get pod -n student12-o name | cut -d/ -f2- | grep mysql)
````


# Creating docker-container of the app 

## BUILD Browse to the docker directory: 

`cd ~/Project/scratch/virtual-insanity/webapp/docker `
Build the container (mind the period, it’s part of the command): 

`docker build -t webapp12:1 .`

## RUN Test the container: 

`docker run –it --rm webapp12:1`

Press CTRL+C and run Bash in the docker: 

`docker run -it --rm webapp12:1 /bin/bashPress `

CTRL+D to exit the container...


##  SHIP

Upload webapp to the local registry 

docker login kube-registry.kube-system.svc.cluster.local User : admin Password : test12345 

`kubectl create secret docker-registry webapp --docker-server=kube-registry.kube-system.svc.cluster.local --docker-username=admin --docker-password=test12345 --docker-email=bosch.peter@icloud.com -n boscp08`

`docker tag webapp12:1 kube-registry.kube-system.svc.cluster.local/webapp12:1docker push kube-registry.kube-system.svc.cluster.local/webapp12:1`


https://10.133.254.59/v2/_catalog 
https:// 10.133.254.59/v2/webapp12/tags/listDeploy 


## DEPLOY webapp to Kubernetes

`kubectl create -f ~/masterclass/webapp/webapp.yaml `

`kubectl get pod -n boscp08`

`kubectl describe pod -n boscp08`
`kubectl logs -n boscp08 $(kubectl get pod -n boscp08 -o name| cut -d/ -f2- | grep webapp) -f`

Zodra het http: // localhost: 8080 in de terminal afdrukt, is uw toepassing actief.
Druk op CTRL + C om te stoppen met het volgen van het logboek. 
App wordt uitgevoerd -

## Tijd om te testen! Vind de load balancer (hint: kijk naar services *)








 
