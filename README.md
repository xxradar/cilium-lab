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
In another terminal2:
```
kubectl create ns hacking
```
```
kubectl run -it --rm -l debug=hacking-mode-n hacking --image xxradar/hackon debug
```

## Applying K8S native network policies
```
kubectl apply -n app-routable-demo -f ./app_routable_demo/network_policies 
```
```
hubble observe -n app-routable-demo --verdict DROPPED -f
```
In terminal2
```
curl  zone1.app-routable-demo.svc.cluster.local/app1
```
```
Aug 27 07:10:58.660: hacking/debug:37254 (ID:59652) <> app-routable-demo/nginx-zone1-5558d47d6b-lpxdf:80 (ID:23472) policy-verdict:none DENIED (TCP Flags: SYN)
Aug 27 07:10:58.660: hacking/debug:37254 (ID:59652) <> app-routable-demo/nginx-zone1-5558d47d6b-lpxdf:80 (ID:23472) Policy denied DROPPED (TCP Flags: SYN)
Aug 27 07:10:59.668: hacking/debug:37254 (ID:59652) <> app-routable-demo/nginx-zone1-5558d47d6b-lpxdf:80 (ID:23472) policy-verdict:none DENIED (TCP Flags: SYN)
Aug 27 07:10:59.668: hacking/debug:37254 (ID:59652) <> app-routable-demo/nginx-zone1-5558d47d6b-lpxdf:80 (ID:23472) Policy denied DROPPED (TCP Flags: SYN)
Aug 27 07:11:01.684: hacking/debug:37254 (ID:59652) <> app-routable-demo/nginx-zone1-5558d47d6b-lpxdf:80 (ID:23472) policy-verdict:none DENIED (TCP Flags: SYN)
Aug 27 07:11:01.684: hacking/debug:37254 (ID:59652) <> app-routable-demo/nginx-zone1-5558d47d6b-lpxdf:80 (ID:23472) Policy denied DROPPED (TCP Flags: SYN)
Aug 27 07:11:05.940: hacking/debug:37254 (ID:59652) <> app-routable-demo/nginx-zone1-5558d47d6b-lpxdf:80 (ID:23472) policy-verdict:none DENIED (TCP Flags: SYN)
Aug 27 07:11:05.940: hacking/debug:37254 (ID:59652) <> app-routable-demo/nginx-zone1-5558d47d6b-lpxdf:80 (ID:23472) Policy denied DROPPED (TCP Flags: SYN)
```
