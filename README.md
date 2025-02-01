# 환경설정
Ubuntu 22.04.5 LTS
Kubernetes 1.32.1
Kubernetes CNI plugin 1.6.2
기준임.

## K8S
https://kubernetes.io/releases/
에서 쿠버네티스 최신버전 확인 할 것.

## CNI Plugin
https://github.com/containernetworking/plugins/releases
에서 플러그인 최신버전 확인 할 것.

가장 중요한 것은 환경설정임.
환경 버전에 아주 예민함.

# 1. 시스템 업데이트 및 필수 패키지 설치
```
sudo apt update
sudo apt install -y apt-transport-https ca-certificates curl
```

# 2. 컨테이너 런타임 설치 (containerd 예제)
```
sudo apt install -y containerd
sudo mkdir -p /etc/containerd
sudo containerd config default | sudo tee /etc/containerd/config.toml
sudo systemctl restart containerd
```

# 3. 스왑 비활성화
```
sudo swapoff -a
sudo sed -i '/ swap / s/^/#/' /etc/fstab
```

# 4. 커널 모듈 및 sysctl 파라미터 설정
## 필요한 커널 모듈 설정
```
cat <<EOF | sudo tee /etc/modules-load.d/k8s.conf
overlay
br_netfilter
EOF
```

```
sudo modprobe overlay
sudo modprobe br_netfilter
```

## sysctl 파라미터 설정
```
cat <<EOF | sudo tee /etc/sysctl.d/k8s.conf
net.bridge.bridge-nf-call-iptables  = 1
net.bridge.bridge-nf-call-ip6tables = 1
net.ipv4.ip_forward                 = 1
EOF
```


```
sudo sysctl --system
```

# 5. Install CNI Plugin
CNI Plugin 버전에 맞는지 확인 할 것  
여기에서는 1.6.2 기준임
```
wget https://github.com/containernetworking/plugins/releases/download/v1.6.2/cni-plugins-linux-amd64-v1.6.2.tgz
mkdir -p /opt/cni/bin
tar Cxzvf /opt/cni/bin cni-plugins-linux-amd64-v1.6.2.tgz
```

권한이 문제라면 su로 접속해서 시도해볼 것
```
sudo passwd root
```
위 명령어로 su 계정 비밀번호 초기화 후 사용가능  

다운 후에는 su 계정 exit


# 6. Forward IPv4 and Configure iptables
```
cat <<EOF | sudo tee /etc/modules-load.d/k8s.conf
overlay
br_netfilter
EOF
sudo modprobe overlay
sudo modprobe br_netfilter

cat <<EOF | sudo tee /etc/sysctl.d/k8s.conf
net.bridge.bridge-nf-call-iptables = 1
net.bridge.bridge-nf-call-ip6tables = 1
net.ipv4.ip_forward = 1
EOF

sudo sysctl --system
sysctl net.bridge.bridge-nf-call-iptables net.bridge.bridge-nf-call-ip6tables net.ipv4.ip_forward
modprobe br_netfilter
sysctl -p /etc/sysctl.conf
```

# 7. Restart containerd and Check the Status
```
sudo systemctl restart containerd && systemctl status containerd
```
containerd가 정상적으로 설치되었는지를 확인


# 8. Install kubeadm, kubelet, and kubectl
쿠버네티스 버전이 1.32.1 기준임  
명령어에서는 1.32로 사용됨
```
sudo apt-get update
sudo apt-get install -y apt-transport-https ca-certificates curl gpg

sudo mkdir -p -m 755 /etc/apt/keyrings

curl -fsSL https://pkgs.k8s.io/core:/stable:/v1.32/deb/Release.key | sudo gpg --dearmor -o /etc/apt/keyrings/kubernetes-apt-keyring.gpg
echo 'deb [signed-by=/etc/apt/keyrings/kubernetes-apt-keyring.gpg] https://pkgs.k8s.io/core:/stable:/v1.32/deb/ /' | sudo tee /etc/apt/sources.list.d/kubernetes.list

sudo apt-get update -y
sudo apt-get install -y kubelet kubeadm kubectl
sudo apt-mark hold kubelet kubeadm kubectl
```

# 10. Initialize the Cluster and Install CNI
```
sudo kubeadm config images pull
```

# 11-1. Master Node 설정
K8S Init
```
sudo kubeadm init
```

단축어 설정
```
mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config
```


# 11-2. Worker Node 설정
Master node에 등록
```
kubeadm join 192.168.122.100:6443 --token zcijug.ye3vrct74itrkesp \
        --discovery-token-ca-cert-hash sha256:e9dd1a0638a5a1aa1850c16f4c9eeaa2e58d03f97fd0403f587c69502570c9cd
```
권한 문제라면 sudo나 su로 실행해보자.


### 출처
- ChatGPT  
- https://www.youtube.com/watch?v=2XlI9qqed04  






