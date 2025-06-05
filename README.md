# kubernetes


### 2025-06
See also
* /Volumes/git/mwk.git/2021/snippets/basic_clusters_still_need_human_sorting/application/snpt_1633771455220_0__Docker__Kubernetes_meetup_.mwk
* /Volumes/trash_numerous/trash/index_mwk/snippets/2024/snippets/snpt_1733944500_2020-12-17_A_pod_must_be_self_c.mwk
* /Volumes/git/computers.git/2024/common/bin/kubectl_create_yaml_programmaticaly.sh


### 
use cases: 
* url biggest image
* url title getting
* Inverse document frequency for files across entire hard drive? (With or without mahout)
    * Can calculate TFIDF in a simple shell script then 

https://blog.andreaskrahl.de/content/images/2020/11/1-QLGlVwQ1wmm6c6TrPg4s-w.png

### TODO
* 2024-03 hello world java http (not rest) servlet (maybe graal vm native compiled)

### 2023-05

#### Part 1: creating a cluster of virtual hardware
See other md file

#### Part 2: deploying an application to the hardware
```
kubectl create deployment mydeployment --image=python --command -- python -m http.server
kubectl expose deployment  mydeployment --port 8000 --type=LoadBalancer --name=my-service
kubectl get services
curl 'http://nuc2020:30805/'
```
### 2023-03
Worked
https://hub.docker.com/r/ubuntu/apache2

	$ kubectl get all --all-namespaces

	NAMESPACE     NAME                                                       READY   STATUS    RESTARTS       AGE
	kube-system   pod/etcd-sarnobat-system-product-name                      1/1     Running   11 (77m ago)   6m7s
	kube-system   pod/kube-apiserver-sarnobat-system-product-name            1/1     Running   10 (75m ago)   6m6s
	kube-system   pod/kube-controller-manager-sarnobat-system-product-name   1/1     Running   12 (75m ago)   6m7s
	kube-system   pod/kube-scheduler-sarnobat-system-product-name            1/1     Running   12 (75m ago)   6m6s

	NAMESPACE   NAME                 TYPE        CLUSTER-IP   EXTERNAL-IP   PORT(S)   AGE
	default     service/kubernetes   ClusterIP   10.96.0.1    <none>        443/TCP   6m5s

	sudo apt-get install containerd

	sudo kubeadm reset
	# The --pod-network-cidr=10.244.0.0/16 option is a requirement for Flannel - don't change that network address!
	sudo kubeadm init --pod-network-cidr=10.244.0.0/16
	sudo sysctl net.bridge.bridge-nf-call-iptables=1
#kubectl edit node sarnobat-system-product-name

	% https://www.itzgeek.com/how-tos/linux/ubuntu-how-tos/install-containerd-on-ubuntu-22-04.html	
	19385  sudo systemctl status containerd
	19386  sudo mkdir -p /etc/containerd/; containerd config default | sudo tee /etc/containerd/config.toml
	19387  sudo sed -i 's/SystemdCgroup \= false/SystemdCgroup \= true/g' /etc/containerd/config.toml
	19388  sudo systemctl restart containerd

	By default, no workloads will run on the master node. You usually want this in a production environment. In my case, since I'm using it for development and testing, I want to allow containers to run on the master node as well. This is done by a process called "tainting" the host.	

	$ kubectl taint nodes --all node-role.kubernetes.io/control-plane-
	node/sarnobat-system-product-name untainted
	$ sudo systemctl restart containerd

	$ curl http://localhost:30080

	<html>
	  <title>Ubuntu apache2 OCI image -- It works!</title>
	  <body>
		<p><h1>Ubuntu apache2 OCI image -- It works!</h1></p>
	  </body>
	</html>
	
