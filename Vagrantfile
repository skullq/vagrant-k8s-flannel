join_file = "/vagrant/join.sh"

Vagrant.configure("2") do |config|
  config.vm.boot_timeout = 3000

  config.vm.define "master" do |master|
    #master.vm.box = "ubuntu/jammy64"
    master.vm.box = "bento/ubuntu-22.04"
    master.vm.network "private_network", ip: "192.168.56.10"
    master.vm.hostname = "master"
    master.vm.provider "vmware_workstation" do |v|
      v.memory = 4096
      v.cpus = 2
      v.gui = true
      v.vmx['vhv.enable'] = 'TRUE'
    end

    master.vm.provision "0", type: "shell", preserve_order: true, privileged: true, inline: <<-EOC
# Kernel Module and Parameter setting
    cat <<-'EOF' >/etc/modules-load.d/kubernetes.conf
br_netfilter
overlay
EOF
sudo modprobe overlay
sudo modprobe br_netfilter
cat <<-'EOF' >/etc/sysctl.d/kubernetes.conf
net.bridge.bridge-nf-call-iptables = 1
net.bridge.bridge-nf-call-ip6tables = 1
net.ipv4.ip_forward = 1
EOF
sudo sysctl --system

# Disable Swap
swapoff -a && sed -i '/swap/s/^/#/' /etc/fstab

# change sources.list
sudo mv /etc/apt/sources.list /etc/apt/sources.list-backup
sudo cp -i /vagrant/sources.list /etc/apt/ 

# Install Containerd
sudo apt-get update
sudo apt-get install -y curl gnupg2 software-properties-common apt-transport-https ca-certificates
sudo curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmour -o /etc/apt/trusted.gpg.d/docker.gpg
sudo add-apt-repository -y "deb [arch=amd64] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable"
sudo apt-get update
sudo apt-get install -y containerd.io
containerd config default | sudo tee /etc/containerd/config.toml >/dev/null 2>&1
sudo sed -i 's/SystemdCgroup \= false/SystemdCgroup \= true/g' /etc/containerd/config.toml
sudo systemctl restart containerd
sudo systemctl enable containerd

# Install Kubernetes
cat <<-'EOF' >/etc/default/kubelet
KUBELET_EXTRA_ARGS=--node-ip=192.168.56.10
EOF
sudo curl -s https://packages.cloud.google.com/apt/doc/apt-key.gpg | gpg --dearmor | sudo dd status=none of=/usr/share/keyrings/kubernetes-archive-keyring.gpg
echo "deb [signed-by=/usr/share/keyrings/kubernetes-archive-keyring.gpg] https://apt.kubernetes.io/ kubernetes-xenial main" | sudo tee /etc/apt/sources.list.d/kubernetes.list
sudo apt-get update
sudo apt-get install -y kubelet kubeadm kubectl
sudo apt-mark hold kubelet kubeadm kubectl
rm -rf #{join_file}
rm -rf /vagrant/.kube
sudo kubeadm init --pod-network-cidr=10.244.0.0/16 --control-plane-endpoint=192.168.56.10 --apiserver-advertise-address=192.168.56.10
sudo kubeadm token create --print-join-command >#{join_file}
chmod +x #{join_file}

mkdir -p $HOME/.kube
sudo cp /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config
cp -R $HOME/.kube /vagrant/.kube
cp -R $HOME/.kube /home/vagrant/.kube
sudo chown -R vagrant:vagrant /home/vagrant/.kube

# Install Helm
curl https://baltocdn.com/helm/signing.asc | gpg --dearmor | sudo tee /usr/share/keyrings/helm.gpg > /dev/null
sudo apt-get install apt-transport-https --yes
echo "deb [arch=$(dpkg --print-architecture) signed-by=/usr/share/keyrings/helm.gpg] https://baltocdn.com/helm/stable/debian/ all main" | sudo tee /etc/apt/sources.list.d/helm-stable-debian.list
sudo apt-get update
sudo apt-get install helm

#CNI Flannel
#kubectl apply -f https://raw.githubusercontent.com/flannel-io/flannel/master/Documentation/kube-flannel.yml
kubectl apply -f /vagrant/kube-flannel.yaml

#CNI Calico
#kubectl apply -f https://docs.projectcalico.org/manifests/calico.yaml

#Setting Env
kubectl completion bash >/etc/bash_completion.d/kubectl
echo 'alias kk=kubectl' >>/home/vagrant/.bashrc

sudo apt-get install -y kubecolor
echo 'alias k=kubecolor' >> /home/vagrant/.bashrc

sudo snap install k9s --channel=latest/stable
echo 'export PATH=$PATH:/snap/k9s/current/bin' >> /home/vagrant/.bashrc
    EOC
  end

  #(1..3).each do |i|
  (1..3).each do |i|
    config.vm.define "worker#{i}" do |worker|
      #worker.vm.box = "ubuntu/jammy64"
      worker.vm.box = "bento/ubuntu-22.04"
      worker.vm.network "private_network", ip: "192.168.56.1#{i}"
      worker.vm.hostname = "worker#{i}"
      worker.vm.provider "vmware_workstation" do |v|
        v.memory = 2048
        v.cpus = 2
        v.gui = true
        v.vmx['vhv.enable'] = 'TRUE'
      end

      worker.vm.provision "0", type: "shell", preserve_order: true, privileged: true, inline: <<-EOC
# Kernel Module and Parameter setting
cat <<-'EOF' >/etc/modules-load.d/kubernetes.conf
br_netfilter
overlay
EOF
sudo modprobe overlay
sudo modprobe br_netfilter
cat <<-'EOF' >/etc/sysctl.d/kubernetes.conf
net.bridge.bridge-nf-call-iptables = 1
net.bridge.bridge-nf-call-ip6tables = 1
net.ipv4.ip_forward = 1
EOF
sudo sysctl --system

# Disable Swap
swapoff -a && sed -i '/swap/s/^/#/' /etc/fstab

# change sources.list
sudo mv /etc/apt/sources.list /etc/apt/sources.list-backup
sudo cp -i /vagrant/sources.list /etc/apt/ 

# Install Containerd
sudo apt-get update
sudo apt-get install -y curl gnupg2 software-properties-common apt-transport-https ca-certificates
sudo curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmour -o /etc/apt/trusted.gpg.d/docker.gpg
sudo add-apt-repository -y "deb [arch=amd64] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable"
sudo apt-get update
sudo apt-get install -y containerd.io
containerd config default | sudo tee /etc/containerd/config.toml >/dev/null 2>&1
sudo sed -i 's/SystemdCgroup \= false/SystemdCgroup \= true/g' /etc/containerd/config.toml
sudo systemctl restart containerd
sudo systemctl enable containerd

# Install Kubernetes
cat <<-'EOF' >/etc/default/kubelet
KUBELET_EXTRA_ARGS=--node-ip=192.168.56.1#{i}
EOF
sudo curl -s https://packages.cloud.google.com/apt/doc/apt-key.gpg | gpg --dearmor | sudo dd status=none of=/usr/share/keyrings/kubernetes-archive-keyring.gpg
echo "deb [signed-by=/usr/share/keyrings/kubernetes-archive-keyring.gpg] https://apt.kubernetes.io/ kubernetes-xenial main" | sudo tee /etc/apt/sources.list.d/kubernetes.list
sudo apt-get update
sudo apt-get install -y kubelet kubeadm kubectl
sudo apt-mark hold kubelet kubeadm kubectl
sudo /bin/bash #{join_file}
      EOC
    end
  end
end