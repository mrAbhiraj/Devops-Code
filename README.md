# 🚀 Ansible Setup Guide — Docker-Based Lab

A step-by-step guide to set up Ansible with a master node and multiple target machines using Docker containers.

---

## 🏗️ Architecture

```
Docker Host (Play with Docker)
├── ansible_master  (Ubuntu container)
├── target1         (Ubuntu container)
└── target2         (Ubuntu container)
```

---

## Part 1 — Create 3 Ubuntu Containers

> Go to [Play with Docker](https://labs.play-with-docker.com/) and create an instance.

```bash
apt-get update && apt install docker.io -y

docker run -itd --name ansible_master ubuntu /bin/bash
docker run -itd --name target1        ubuntu /bin/bash
docker run -itd --name target2        ubuntu /bin/bash

# Verify containers are running
docker ps
```

---

## Part 2 — Install Ansible on `ansible_master`

```bash
docker exec -it ansible_master bash

apt update
apt install python-is-python3 vim iputils-ping openssh-client -y

# When prompted: Geographical area → 5 (Asia), City → 44 (Kolkata or your city)
apt install software-properties-common -y
add-apt-repository --yes --update ppa:ansible/ansible
apt install ansible -y

# Verify installation
ansible --version
```

---

## Part 3 — Setup Target Machines

> Repeat **Steps 5–9** for each target (`target1`, `target2`, ...).

### Step 1 — Enter target container

```bash
docker exec -it target1 bash
```

### Step 2 — Install dependencies & SSH

```bash
apt update
apt install vim python-is-python3 iputils-ping openssh-client -y
apt-get install openssh-client openssh-server -y
```

### Step 3 — Allow root login via SSH

```bash
cd /etc/ssh
vi sshd_config
```

Uncomment and set the following:

```
PermitRootLogin yes
PasswordAuthentication yes
```

### Step 4 — Start SSH service

```bash
service ssh start
service ssh status
```

### Step 5 — Set root password

```bash
passwd root
# Enter: Admin (twice)
```

> ✅ Repeat **Part 3** for `target2` and any additional targets.

---

## Part 4 — Find Target Container IPs

> Exit back to the Docker host node and run:

```bash
sudo docker inspect target1
sudo docker inspect target2
```

Look for `"IPAddress"` in the output, e.g.:

```
"IPAddress": "172.17.0.3"
```

---

## Part 5 — Configure `ansible_master` for SSH Connectivity

### Step 1 — Enter master container

```bash
docker exec -it ansible_master bash
```

### Step 2 — Add target IPs to Ansible hosts file

```bash
cd /etc/ansible/
vi hosts
```

Add target IPs:

```
172.17.0.3
172.17.0.4
```

### Step 3 — Verify network connectivity

```bash
ping 172.17.0.3
```

---

## Part 6 — Setup SSH Key-Based Authentication

### Step 1 — Generate SSH key on `ansible_master`

```bash
ssh-keygen
# Press Enter 3 times (accept default path + empty passphrase x2)
```

### Step 2 — Copy SSH key to each target

```bash
ssh-copy-id root@172.17.0.3
ssh-copy-id root@172.17.0.4
# Type 'yes' when prompted
```

### Step 3 — Verify SSH connection

```bash
ssh root@172.17.0.3
```

---

## Part 7 — Create & Run an Ansible Playbook

### Step 1 — Create the playbook

```bash
cd /etc/ansible
vi installnginx.yaml
```

```yaml
---
- hosts: all
  tasks:
    - name: ensure nginx is at the latest version
      apt:
        name: nginx
        state: latest
```

### Step 2 — Run the playbook

```bash
ansible-playbook installnginx.yaml
```

### Step 3 — Verify nginx on target

```bash
ssh root@172.17.0.3
service nginx status
```

---

## 📋 Quick Reference

| Container       | Role          | Default IP     |
|----------------|---------------|----------------|
| ansible_master | Control Node  | 172.17.0.2     |
| target1        | Managed Node  | 172.17.0.3     |
| target2        | Managed Node  | 172.17.0.4     |

---

## ⚠️ Notes

- This setup uses **root + password** auth for simplicity in a lab environment. Use SSH keys and non-root users in production.
- Container IPs may vary — always verify with `docker inspect`.
- `Play with Docker` sessions expire after **4 hours**.
