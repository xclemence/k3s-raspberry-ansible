# k3s raspberry ansible

This repository contains Ansible scripts to deploy a k3s on Raspberry PI.

Scripts are based on `k3s.io` scripts, you can found the original scripts [here](https://github.com/k3s-io/k3s-ansible)

## Raspberry preparation

### Install Raspberry  Pi
- Download latest [Raspberry Pi Imager](https://www.raspberrypi.org/software/)
- Download latest version of [Raspberry Pi OS **64 bit** (beta version)](https://downloads.raspberrypi.org/raspios_lite_arm64/images/)
- Mount OS image with custom image option.

### Enable SSH connection
- Add empty ssh file on the boot partition

### Change hostname

```shell
sudo raspi-config
```

- 1 - System Options
- S4 Hostname 

### Set SSH key for connection

- Run powershell **as administrator**

- Generate public/private ecdsa key pair.

```powershell
ssh-keygen -t ecdsa -b 521
```

- Apply public key on all servers

```powershell
$PUBKEYPATH="$HOME\.ssh\id_ecdsa.pub" # public key localisation
$HOSTS= @("pi@192.168.0.150","pi@192.168.0.151","pi@192.168.0.155") # servers list

$HOSTS | Foreach-Object { 
    $pubKey=(Get-Content "$PUBKEYPATH" | Out-String); ssh $_ "mkdir -p ~/.ssh && chmod 700 ~/.ssh && echo '${pubKey}' >> ~/.ssh/authorized_keys && chmod 600 ~/.ssh/authorized_keys" 
}
```

- Initialise ssh agent and add current key

```powershell
Set-Service ssh-agent -StartupType Automatic
Start-Service ssh-agent

ssh-add
```

## Ansible Scripts

## Inventory

- master: Kubernetes master
- node: Kubernetes worker node
- tools: server with tools to manage the Kubernetes server (kubectl, helm)

## Playbook

Tools/plugins installed:
- k3s
- Kubernetes dashboard (Helm)
- Longhorn (Helm)
- Ingress nginx (Helm)

Ansible galaxy collections required:
- ommunity.general
- ommunity.kubernetes

### Install the k3s

```shell
ansible-playbook site.yml -i inventory/hosts.ini
```

### uninstall the k3s
```shell
ansible-playbook reset.yml -i inventory/hosts.ini
```

## Reminders

### Dashboard access
- run 
```powershell
kubectl proxy
```
- open: http://localhost:8001/api/v1/namespaces/dashboard/services/https:kubernetes-dashboard:https/proxy/

### Find kubernetes dashboard secrets token

```powershell
kubectl -n dashboard describe secret $(kubectl -n dashboard get secret | where-object { $_.startswith('dashboard-token') } | ForEach-Object { $_ -Split '\s+' } | Select -First 1)
```

