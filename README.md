# This is a Kubernetes cluster deployment experiment for data persistent storage test.

Step 1: Setup and Deploy Kubernetes Cluster
1, install Kubernetes cluster
   1) There are three servers for K8S cluster. These servers are running on AWS EC2.
      K8S_Master: 18.136.119.223/172.31.25.221
      K8S_Node1:  52.221.200.78/172.31.30.7
      K8S_Node2:  54.254.229.151/172.31.27.212
   2) install Ansible tool on local machine.
      brew install ansible
   3) initiate and install kubernetes master and node servers through ansible playbook.
      ansible-playbook -i hosts ~/kube-cluster/kube-dependencies.yml
   4)
   






Step 2: Install Helm package Management



Step 3: Install Wordpress and MariaDB for persistent storage test





