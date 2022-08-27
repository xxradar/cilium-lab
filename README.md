# cilium-lab

## Lab setup
Kubeadm K8S cluster v1.25<br> 
Cilium v1.12<br>
#### app-routable-demo
See [app-routable-demo](https://github.com/xxradar/app_routable_demo) for more info<br> 
```
git clone https://github.com/xxradar/app_routable_demo.git
cd ./app_routable_demo
./setup.sh
watch kubectl get po -n app-routable-demo
```

## Observing network flows
```
hubble observe -n app-routable-demo
```
```
Aug 27 07:00:07.474: app-routable-demo/nginx-zone5-b4df4559d-6rr97:41366 (ID:34491) -> app-routable-demo/echoserver-2-deployment-56f8846f4-vsm5l:80 (ID:3633) to-overlay FORWARDED (TCP Flags: ACK, FIN)
Aug 27 07:00:07.474: app-routable-demo/nginx-zone5-b4df4559d-6rr97:41366 (ID:34491) <- app-routable-demo/echoserver-2-deployment-56f8846f4-vsm5l:80 (ID:3633) to-overlay FORWARDED (TCP Flags: ACK, FIN)
Aug 27 07:00:07.474: app-routable-demo/nginx-zone2-6f46f7849f-d2dn7:36962 (ID:16965) <- app-routable-demo/nginx-zone5-b4df4559d-6rr97:80 (ID:34491) to-overlay FORWARDED (TCP Flags: ACK, PSH)
Aug 27 07:00:07.474: app-routable-demo/nginx-zone5-b4df4559d-6rr97:41366 (ID:34491) <- app-routable-demo/echoserver-2-deployment-56f8846f4-vsm5l:80 (ID:3633) to-endpoint FORWARDED (TCP Flags: ACK, FIN)
Aug 27 07:00:07.474: app-routable-demo/nginx-zone5-b4df4559d-6rr97:41366 (ID:34491) -> app-routable-demo/echoserver-2-deployment-56f8846f4-vsm5l:80 (ID:3633) to-overlay FORWARDED (TCP Flags: ACK)
Aug 27 07:00:07.474: app-routable-demo/nginx-zone5-b4df4559d-6rr97:41366 (ID:34491) -> app-routable-demo/echoserver-2-deployment-56f8846f4-vsm5l:80 (ID:3633) to-endpoint FORWARDED (TCP Flags: ACK, FIN)
```
## Create a testing pod
```
kubectl create ns hacking
```
```
kubectl run -it --rm -l debug=hacking-mode --image xxradar/hackon debug
```

## Applying K8S native network policies
```
kubectl apply -n app-routable-demo -f ./app_routable_demo/network_policies 
```
