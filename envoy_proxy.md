## some notes

```
apiVersion: cilium.io/v2
kind: CiliumNetworkPolicy
metadata:
  name: proxy-policy
spec:
  endpointSelector: {}
  egress:
    - toEntities:
        - cluster
    - toEndpoints:
        - matchLabels:
            io.kubernetes.pod.namespace: kube-system
            k8s-app: kube-dns
      toPorts:
        - ports:
            - port: "53"
              protocol: ANY
          rules:
            dns:
              - matchPattern: "*"
    - toFQDNs:
        - matchPattern: "*.github.com"
      toPorts:
        - ports:
            - port: "80"
              protocol: TCP
            - port: "443"
              protocol: TCP
          listener:
            envoyConfig:
              kind: "CiliumEnvoyConfig"
              name: "proxy-envoy"
            name: "proxy-listener"
```
```
apiVersion: cilium.io/v2
kind: CiliumEnvoyConfig
metadata:
  name: proxy-envoy
spec:
  resources:
  - "@type": type.googleapis.com/envoy.config.listener.v3.Listener
    name: proxy-listener
    filter_chains:
    - filters:
      - name: envoy.filters.network.tcp_proxy
        typed_config:
          "@type": type.googleapis.com/envoy.extensions.filters.network.tcp_proxy.v3.TcpProxy
          stat_prefix: tcp_stats
          tunneling_config:
            hostname: "%DOWNSTREAM_LOCAL_ADDRESS%"
          cluster: proxy-cluster
  - "@type": type.googleapis.com/envoy.config.cluster.v3.Cluster
    name: proxy-cluster
    connect_timeout: 55s
    type: STATIC
    load_assignment:
      cluster_name: proxy-cluster
      endpoints:
      - lb_endpoints:
        - endpoint:
            address:
              socket_address:
                address: 172.18.0.5
                port_value: 3128
```
