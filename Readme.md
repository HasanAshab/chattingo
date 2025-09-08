## Setup VPS

### 1. Setup Password-less SSH to VPS
On your Jenkins server or agent node, run:

```bash
ssh-keygen -t ed25519 -C "chattingo-vps" -f ~/.ssh/chattingo_vps
```

This gives you:

* Private key → `~/.ssh/chattingo_vps`
* Public key → `~/.ssh/chattingo_vps.pub`

Now copy the public key:
```bash
cat ~/.ssh/chattingo_vps.pub
```

Then Go to _Hostinger Dashboard > Settings > SSH Keys > Add SSH Key_ And paste the public key there.
I repeat the **public** key.

Now SSH to your VPS.
```bash
ssh -i ~/.ssh/chattingo_vps root@<your-vps-ip>
```

---


### 2. Install Docker & Docker Compose

```bash
apt update && apt upgrade -y
apt install -y docker.io docker-compose
systemctl enable docker
systemctl start docker
```

Verify:

```bash
docker --version
docker-compose --version
```

---

### 3. Install Git (if not present)
On my VPS there was already git installed.

But if you don't have git installed, you can install it using the following command:
```bash
apt install -y git
```

---

### 4. Clone the repository
Now we have to clone [chatingo-compose](https://github.com/HasanAshab/chattingo-compose) repo (not the chatingo repo)

```bash
git clone https://github.com/HasanAshab/chattingo-compose.git
```

---
### 5. Run Docker Compose
```bash
# go to the cloned directory
cd chattingo-compose

# run setup script with your mysql password
chmod +x setup.sh
./setup.sh <your db password>

# run docker compose
docker-compose up -d
```
