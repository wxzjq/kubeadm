Kubernetes installation and deploy wordpress for test

Deploy files:

	├── README.md
	├── hosts
	├── kube-dependencies.yaml
	├── kube-master.yaml
	├── kube-worker.yaml
	├── mariadb-pv.yaml
	├── mariadb-pvc.yaml
	├── tree.txt
	├── wordpress-pv.yaml
	└── wordpress-pvc.yaml

1, kubernetes cluster installation:

	1) 	install docker and related package on Master and workers.
		ansible-playbook -i hosts kube-dependencies.yaml

	2) 	configure Kubernetes master server.
		ansible-playbook -i hosts kube-master.yaml

	3) 	configure Kubernetes worker servers.
		ansible-playbook -i hosts kube-worker.yaml

	4) 	check Kubernetes cluster nodes.
		ubuntu@ip-172-31-25-221:~$ kubectl get nodes

		NAME               STATUS   ROLES    AGE     VERSION
		ip-172-31-25-221   Ready    master   6m45s   v1.14.0
		ip-172-31-27-212   Ready    <none>   4m14s   v1.14.0
		ip-172-31-30-7     Ready    <none>   4m14s   v1.14.0

	5)  Kubernetes cluster is ready for services.


2, Helm package management installation:

	1) install helm client
	   curl https://raw.githubusercontent.com/kubernetes/helm/master/scripts/get > install-helm.sh
	   chmod u+x install-helm.sh
	   ./install-helm.sh
	   helm init

	2) install Tiller server
	   kubectl -n kube-system create serviceaccount tiller
	   
	3) Check tiller running status:
	    ubuntu@ip-172-31-25-221:~$ kubectl get pods --namespace kube-system
	   	
	   	NAME                                       READY   STATUS    RESTARTS   AGE
		coredns-fb8b8dccf-4c6kz                    1/1     Running   0          12m
		coredns-fb8b8dccf-nbc2k                    1/1     Running   0          12m
		etcd-ip-172-31-25-221                      1/1     Running   0          11m
		kube-apiserver-ip-172-31-25-221            1/1     Running   0          12m
		kube-controller-manager-ip-172-31-25-221   1/1     Running   0          11m
		kube-flannel-ds-amd64-24zfx                1/1     Running   0          10m
		kube-flannel-ds-amd64-b2fz4                1/1     Running   0          12m
		kube-flannel-ds-amd64-plxc6                1/1     Running   0          10m
		kube-proxy-2c7q8                           1/1     Running   0          10m
		kube-proxy-p7wkp                           1/1     Running   0          12m
		kube-proxy-wtrmn                           1/1     Running   0          10m
		kube-scheduler-ip-172-31-25-221            1/1     Running   0          12m
		tiller-deploy-765dcb8745-c9rvz             1/1     Running   0          92s

	4)  Give tiller permission:
		kubectl --namespace kube-system create serviceaccount tiller
		kubectl create clusterrolebinding tiller-cluster-rule --clusterrole=cluster-admin --serviceaccount=kube-system:tiller
	 	kubectl --namespace kube-system patch deploy tiller-deploy -p '{"spec":{"template":{"spec":{"serviceAccount":"tiller"}}}}' 

	5)  Update helm repo:
	    helm repo update

	6)  helm installation finished.

3, Install Wordpress and MariaDB through Helm, and test persistent claim on Kubernetes
	
	1) 	Create 2 EBS block on AWS, one is for Wordpress persistent storage, another is for MariaDB persistent storage.

		wordpress: 
		aws ec2 create-volume --availability-zone ap-southeast-1a --size 20 --volume-type gp2

  		MariaDB:
  		aws ec2 create-volume --availability-zone ap-southeast-1a --size 1 --volume-type gp2

  	2) 	Create wordpress PV&PVC through kubectl
  	   	kubectl create -f wordpress-pv.yaml
  	   	kubectl create -f wordpress-pvc.yaml

  	3) 	Create MariaDB PV&PVC through kubectl
	   	kubectl create -f mariadb-pv.yaml
  	   	kubectl create -f mariadb-pvc.yaml

  	4) 	Check PV&PBC Status

  		ubuntu@ip-172-31-25-221:~$ kubectl get pv,pvc

  		NAME                            CAPACITY   ACCESS MODES   RECLAIM POLICY   STATUS   CLAIM                              STORAGECLASS   REASON   AGE
		persistentvolume/mariadb-pv     1Gi        RWO,ROX        Retain           Bound    default/data-wordpress-mariadb-0                           89s
		persistentvolume/wordpress-pv   10Gi       RWO,ROX        Retain           Bound    default/wordpress-wordpress                                2m20s

		NAME                                             STATUS   VOLUME         CAPACITY   ACCESS MODES   STORAGECLASS   AGE
		persistentvolumeclaim/data-wordpress-mariadb-0   Bound    mariadb-pv     1Gi        RWO,ROX                       84s
		persistentvolumeclaim/wordpress-wordpress        Bound    wordpress-pv   10Gi       RWO,ROX 

	5) 	Launch Wordpress through Helm client

		helm install --name wordpress --set wordpressUsername=admin,wordpressPassword=adminpassword,mariadb.mariadbRootPassword=secretpassword,persistence.existingClaim=wordpress-wordpress,allowEmptyPassword=false stable/wordpress

	6) 	Check Wordpress Status

	   	ubuntu@ip-172-31-25-221:~$ kubectl get svc

	   	NAME                  TYPE           CLUSTER-IP      EXTERNAL-IP   PORT(S)                      AGE
		kubernetes            ClusterIP      10.96.0.1       <none>        443/TCP                      38m
		wordpress-mariadb     ClusterIP      10.99.188.48    <none>        3306/TCP                     18s
		wordpress-wordpress   LoadBalancer   10.99.161.140   <pending>     80:30219/TCP,443:30137/TCP   18s









