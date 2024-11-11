# How to set up the K3s cluster
---
## Setting up server node (master node in k8s)

We will use Ubuntu 24 to configure the k3s server node

run the below command to use the privilege command.
```bash
sudo -s
```

install k3s server with user mode
```bash
curl -sfL https://get.k3s.io | K3S_KUBECONFIG_MODE="644" sh -s
```

move the token for cluster authentication to .kube folder
```bash
sudo cat /var/lib/rancher/k3s/server/node-token > ~/.kube/node_token
```

print out the token and save it for connecting agent nodes to the cluster later
```bash
cat ~/.kube/node_token
```

print out the ip address of the server node by running this command
```bash
hostname -I
```
depends on your network configuration, please re-check which ip-address is the right one for you network


extract kube-config file for accessing the kubernetes cluster at .kube/config
```bash
sudo kubectl config view --raw > ~/.kube/config
```

In this project we will use nginx ingress so we have to disable the default ingress of k3s which is traefik ingress.

run
```bash
sudo nano /etc/systemd/system/k3s.service
```

write this option to the last line
```bash
--disable=traefik \
```

The file should look like this
```bash
ExecStart=/usr/local/bin/k3s \
    server \
    --disable=traefik \
```

Create traefik skip file
```bash
sudo touch /var/lib/rancher/k3s/server/manifests/traefik.yaml.skip
```

Now restart the k3s server to disable traefik ingress
```bash
sudo systemctl restart k3s
```

Install Helm for deploying nginx ingress
```bash
cd
curl -fsSL -o get_helm.sh https://raw.githubusercontent.com/helm/helm/main/scripts/get-helm-3
chmod 700 get_helm.sh
./get_helm.sh
```

Install nginx ingress controller for this project
```bash
helm repo add nginx-stable https://helm.nginx.com/stable
helm repo update
helm install nginx-ingress nginx-stable/nginx-ingress --set rbac.create=true
```

The server node is now ready for other agent nodes to join !!

##### Optional set up

If you don't want pods to be created at server node, you can taint the server node using this command
```bash
kubectl taint nodes <NODE_NAME> node-role.kubernetes.io/master=:NoSchedule
```
---
## Setting up agent node (worker node in k8s)

After ssh to the raspberrypi,
you can change the hostname by running this command (Optional)
```bash
sudo raspi-config
```

Install the k3s-agent, use the token and ip address you have printed from the server node
```bash
curl -sfL https://get.k3s.io | K3S_TOKEN="<Token From Main Cluster>" K3S_URL="https://<Main Cluster IP>:6443" K3S_NODE_NAME="<Node Name>" sh -
```

You can check whether the node is in the cluster by running this command in `server node`
```bash
kubectl get nodes
```

## How to deploy the application

After every k3s-agents has connected to the cluster

pull this repository at the master node.
```bash
git pull https://github.com/Iammrteapot-learning/sds-k3s.git
```
run this command
```bash
cd sds-k3s/chat
kubectl apply -f .
```

## Congratulation! The application is now ready!

You can verify that the pods are running in the raspberrypi by running this command
```bash
kubectl get pods -o wide
```