# cilium-lab

## Lab setup
Kubeadm K8S cluster v1.25<br> 
Cilium v1.12<br>
[app-routable-demo](https://github.com/xxradar/app_routable_demo)<br> 
```
git clone https://github.com/xxradar/app_routable_demo.git
cd ./app_routable_demo
./setup.sh
watch kubectl get po -n app-routable-demo
```

## Observing network flows
```
hubble observe -n app-routable-dempo
```

## Applying K8S native network policies
