Use ansible to install kubernetes cluster.

首先必须先在节点的同级网络中建一个代理，否则Docker拉取google镜像的时候会失败，强行修改不是省心的好办法。

同时注意master节点和worker节点访问同级网络 172.16.0.0/16 以及 172.18.0.0/16 以及docker registry的时候无需代理

代理建好后，需要建一个docker_insecure_registry，这里是172.18.31.28，用harbor搭的

随后要把 image: quay.io/coreos/flannel:v0.12.0-amd64 的镜像推到harbor上去的:
image: {{ docker_insecure_registry }}/library/quay.io/coreos/flannel:v0.12.0-amd64"

否则安装flanel网络插件，docker去拉flanel镜像的时候9成9失败

vars.yml变量如下：
```
docker_http_proxy: http://172.16.8.1:3128
docker_https_proxy: http://172.16.8.1:3128
docker_no_proxy: "localhost,127.0.0.0/8,172.16.0.0/16,172.18.0.0/16,172.18.31.28"
docker_insecure_registry: 172.18.31.28

k8s_http_proxy: http://172.16.8.1:3128

k8s_pod_network:
  cni: 'flannel'
  cidr: '10.244.0.0/20'
  svc: '192.168.244.0/20'
  localdns: 'cluster.local'
```
hosts清单中标明每个节点的身份：
```
[k8s_t:vars]
ansible_ssh_user="root"　　　　　　　
[k8s_t]
172.18.31.50 k8s_role=master
172.18.31.51 k8s_role=worker
172.18.31.52 k8s_role=worker
```
执行ansible -k -i hosts main.yml后输入密码即可安装。 

同时roles k8s下的matallb-setup.yml为安装matallb预留了yml，可按需安装。


