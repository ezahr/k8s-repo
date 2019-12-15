# k8s-repo  spiekbriefje


Important info: At the start of the presentation you will be assigned a student number.  L

Vm setup: Kubernetes Linux DisciplVM 5.0.0-1027-azure #29~18.04.1-Ubuntu 
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
 
kubctl get namespaces 
kubectl get pod –-all-namespaces –o wide 
kubectl get pod –n kube-system 
kubectl get service --all-namespaces –o wide 
kubectl get service -n kube-system 
kubectl describe pod -n kube-system coredns-... 
kubectl describe service -n kube-system coredns 

Beetje geavanceerder ..

`kubectl describe pod -n kube-system $(kubectl get pod -n kube-system -o name| cut -d/ -f2- |grep dns)`


 dashboard under contruction
 https://discipl.westeurope.cloudapp.azure.com 
 
 
wijzig  the mysql configfile 
Browse to ~/masterclass/mysql 
Generate the hash for your password (choose your own password) 
`echo -n "ThisIsCool" |base64 `
Save the output  (remember your password) 
Open mysql.yaml with an editor and modify: 

data:
  password: `insert your generated base64 hash`

Save the file!  

## Start mysql in Kubernetes 
kubectl create -f mysql.yaml kubectl get pod -n boscp08  (where -n is the name of your namespace)
kubectl get service -n student12 
Create your database (Modify the password if you picked your own): 
cat start.sql| kubectl -n student12 exec $(kubectl get pod -n student12 -o name | cut -d/ -f2- | grep mysql) -ti -- mysql -u root --password=ThisIsCool 

If you got the output “unable to use a TTY” you have been successful. kubectl -n student12 logs $(kubectl get pod -n student12-o name | cut -d/ -f2- | grep mysql)



# Creating docker-container of the app 
## Browse to the docker directory: 

`cd ~/Project/scratch/virtual-insanity/webapp/docker `
Build the container (mind the period, it’s part of the command): 
`docker build -t webapp12:1 .`

## Test the container: 

`docker run –it --rm webapp12:1`

Press CTRL+C and run Bash in the docker: 

`docker run -it --rm webapp12:1 /bin/bashPress `

CTRL+D to exit the container...





 
