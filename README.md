# pihole-vpn-server
Helm chart deploying a VPN server embedding a PiHole instance. The VPN server K8s service is deployed as NodePort, meaning a few steps need to be performed on the host to make the service globally accessible.

### Things to know
Client-to-client connections connecting on the same VPN server is active.

# How to install
```bash
$ helm install my-vpn-server -f values.yaml
```

## Values description
```yaml
# the external hostname or ip address, in default case, Node ip address
externalHost:

# ip address of the VPN gateway
vpnGateway: 192.168.255.1

statefulset:
  replicas: 1
  containers:
    openvpn:
      resources:
        requests:
          memory: 128Mi
          cpu: 100m
        limits:
          memory: 128Mi
          cpu: 500m

    pihole:
      env:
        TZ: Europe/Berlin
        FTLCONF_webserver_api_password: cat playing with moose
      resources:
        requests:
          memory: 128Mi
          cpu: 100m
        limits:
          memory: 256Mi
          cpu: 500m

service:
  ports:
    - protocol: UDP
      port: 1194
      targetPort: 1194
      nodePort: 32432
```
