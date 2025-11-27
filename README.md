# pihole-vpn-server
Helm chart deploying a VPN server embedding a PiHole instance.

### Things to know
* The pihole-vpn-server service is by default deployed as NodePort.
* A few steps need to be performed on the host to make the service globally accessible, depending on your setup.
* Users have acccess to the PiHole panel generally at `192.168.255.1`, which is protected by an admin password.
* Default ads lists are loaded at startup of the pihole-vpn-server instance, being persistnent accross restarts.  
* Client-to-client connections are effective for all clients connecting on the same pihole-vpn-server instance.

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
