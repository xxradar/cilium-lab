# cilium-lab

## Lab setup
Kubeadm K8S cluster v1.25<br> 
#### Cilium v1.12.1
```
cilium install \
--helm-set ingressController.enabled=true \
--helm-set securityContext.privileged=true \
--kube-proxy-replacement=strict \
--version 1.12.1
```
#### Install demo app-routable-demo
See [app-routable-demo](https://github.com/xxradar/app_routable_demo) for more info<br> 
```
git clone https://github.com/xxradar/app_routable_demo.git
cd ./app_routable_demo
./setup.sh
watch kubectl get po -n app-routable-demo
```

## Observing network flows
```
hubble observe -n app-routable-demo -f 
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
```
curl  zone1.app-routable-demo.svc.cluster.local/app1
...
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

## Create an cilium ingress resource
First delete the network policies in place
```
kubectl delete netpol -f app_routable_demo/network_policies/
```
```
kubectl apply -f - <<EOF
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: app1-ingress
  namespace: app-routable-demo
spec:
  ingressClassName: cilium
  rules:
  - http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: zone1
            port:
              number: 80
EOF
```
```
kubectl get ingress -n app-routable-demo
```
cilium ingress will create a SVC of type LoadBalancer for each ingress
```
kubectl get svc -n app-routable-demo
```
Note the NodePort for some local testing.
```
hubble observe  --from-identity ingress  -f
```
In other terminal2:
```
curl 127.0.0.1:32665/app1
```
Observe the output
```
Aug 27 08:40:09.757: 10.0.2.79:35475 (ingress) <> app-routable-demo/nginx-zone1-5558d47d6b-lpxdf:80 (ID:23472) to-overlay FORWARDED (TCP Flags: SYN)
Aug 27 08:40:09.758: 10.0.2.79:35475 (ingress) -> app-routable-demo/nginx-zone1-5558d47d6b-lpxdf:80 (ID:23472) to-endpoint FORWARDED (TCP Flags: SYN)
Aug 27 08:40:09.758: 10.0.2.79:35475 (ingress) <> app-routable-demo/nginx-zone1-5558d47d6b-lpxdf:80 (ID:23472) to-overlay FORWARDED (TCP Flags: ACK)
Aug 27 08:40:09.758: 10.0.2.79:35475 (ingress) -> app-routable-demo/nginx-zone1-5558d47d6b-lpxdf:80 (ID:23472) to-endpoint FORWARDED (TCP Flags: ACK)
Aug 27 08:40:09.758: 10.0.2.79:35475 (ingress) <> app-routable-demo/nginx-zone1-5558d47d6b-lpxdf:80 (ID:23472) to-overlay FORWARDED (TCP Flags: ACK, PSH)
Aug 27 08:40:09.758: 10.0.2.79:35475 (ingress) -> app-routable-demo/nginx-zone1-5558d47d6b-lpxdf:80 (ID:23472) to-endpoint FORWARDED (TCP Flags: ACK, PSH)
Aug 27 08:40:09.772: 10.0.2.79:35475 (ingress) <> app-routable-demo/nginx-zone1-5558d47d6b-lpxdf:80 (ID:23472) to-overlay FORWARDED (TCP Flags: ACK)
```
## Fixing the network policies with CiliumNetworkPolicy
```
kubectl apply -n app-routable-demo -f ./app_routable_demo/network_policies 
```
```
hubble observe  --from-identity ingress  -f
```
In other terminal2:
```
curl 127.0.0.1:32665/app1
```
Observe the output
```
Aug 27 08:47:22.162: 10.0.2.79:46485 (ingress) <> app-routable-demo/nginx-zone1-5558d47d6b-lpxdf:80 (ID:23472) policy-verdict:none DENIED (TCP Flags: SYN)
Aug 27 08:47:22.162: 10.0.2.79:46485 (ingress) <> app-routable-demo/nginx-zone1-5558d47d6b-lpxdf:80 (ID:23472) Policy denied DROPPED (TCP Flags: SYN)
Aug 27 08:47:23.171: 10.0.2.79:46485 (ingress) <> app-routable-demo/nginx-zone1-5558d47d6b-lpxdf:80 (ID:23472) to-overlay FORWARDED (TCP Flags: SYN)
Aug 27 08:47:23.172: 10.0.2.79:46485 (ingress) <> app-routable-demo/nginx-zone1-5558d47d6b-lpxdf:80 (ID:23472) policy-verdict:none DENIED (TCP Flags: SYN)
Aug 27 08:47:23.172: 10.0.2.79:46485 (ingress) <> app-routable-demo/nginx-zone1-5558d47d6b-lpxdf:80 (ID:23472) Policy denied DROPPED (TCP Flags: SYN)
```
```
kubectl apply -n app-routable-demo -f - <<EOF
apiVersion: "cilium.io/v2"
kind: CiliumNetworkPolicy
metadata:
  name: "cilium-ingress"
spec:
  endpointSelector:
    matchLabels:
      app: nginx-zone1
  ingress:
    - fromEntities:
      - ingress
EOF
```
```
Aug 27 08:49:44.416: 10.0.2.79:39309 (ingress) -> app-routable-demo/nginx-zone1-5558d47d6b-lpxdf:80 (ID:23472) policy-verdict:L3-Only ALLOWED (TCP Flags: SYN)
Aug 27 08:49:44.416: 10.0.2.79:39309 (ingress) -> app-routable-demo/nginx-zone1-5558d47d6b-lpxdf:80 (ID:23472) to-endpoint FORWARDED (TCP Flags: SYN)
Aug 27 08:49:44.416: 10.0.2.79:39309 (ingress) <> app-routable-demo/nginx-zone1-5558d47d6b-lpxdf:80 (ID:23472) to-overlay FORWARDED (TCP Flags: ACK)
Aug 27 08:49:44.416: 10.0.2.79:39309 (ingress) <> app-routable-demo/nginx-zone1-5558d47d6b-lpxdf:80 (ID:23472) to-overlay FORWARDED (TCP Flags: ACK, PSH)
Aug 27 08:49:44.416: 10.0.2.79:39309 (ingress) -> app-routable-demo/nginx-zone1-5558d47d6b-lpxdf:80 (ID:23472) to-endpoint FORWARDED (TCP Flags: ACK)
Aug 27 08:49:44.417: 10.0.2.79:39309 (ingress) -> app-routable-demo/nginx-zone1-5558d47d6b-lpxdf:80 (ID:23472) to-endpoint FORWARDED (TCP Flags: ACK, PSH)
```
## DNS visibility

## L7 visibility


