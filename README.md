# pihole-vpn-server
Helm chart deploying a VPN server embedding a PiHole instance.

### Things to know
* The pihole-vpn-server service is by default deployed as NodePort.
* A few steps need to be performed on the host to make the service globally accessible, depending on your setup.
* Users have acccess to the PiHole panel generally at `192.168.255.1`, which is protected by an admin password.
* Default ads lists are loaded at startup of the pihole-vpn-server instance, being persistent accross restarts.  
* Client-to-client connections are effective for all clients connecting on the same pihole-vpn-server instance.

# How to install
```bash
$ helm install my-vpn-server -f values.yaml
```

## Values description
```yaml
# the external hostname or ip address, in default case, host ip address
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

## How to add a VPN user

Assuming you have access to the host system as Helm would require, we can use Ansible to automate VPN user creation.
These following Ansible tasks trigger commands executed inside the VPN server pod.
```yaml
- name: Add VPN server rules
  hosts: host1
  environment:
    PATH: "{{ ansible_env.PATH }}:/usr/sbin/"
  vars:
    new_vpn_user: "username_created"
    vpn_pod_name: "ovpn-pihole-pod"
    namespace: "vpn-ns"
  tasks:
    - name: Create a random password
      ansible.builtin.shell: |
        tr -dc 'A-Za-z0-9' < /dev/urandom | head -c 15
      changed_when: true
      register: random_password

    - name: Print the random password
      ansible.builtin.debug:
        msg: "The random password is {{ random_password.stdout }}"

    - name: Generate OpenVPN client key with password using expect
      ansible.builtin.expect:
        command: >
          kubectl -n {{ namespace }} exec -it {{ vpn_pod_name }} -c openvpn --
          easyrsa build-client-full {{ new_vpn_user }}
        responses:
          "Enter PEM pass phrase:": "{{ random_password.stdout }}"
          "Verifying - Enter PEM pass phrase:": "{{ random_password.stdout }}"
        echo: yes
      register: openvpn_output

    - name: Build OpenVPN client file
      ansible.builtin.command: >
        kubectl -n {{ namespace }} exec -it {{ vpn_pod_name }} -c openvpn -- ovpn_getclient {{ new_vpn_user }}
      register: build_client_result
      changed_when: "'Generating' in build_client_result.stdout"

    - name: Save OpenVPN client build output to local file
      ansible.builtin.copy:
        content: "{{ build_client_result.stdout }}"
        dest: "./openvpn_client_build_output_{{ new_vpn_user }}.ovpn"
        mode: '0644'
      delegate_to: localhost
```

