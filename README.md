# 🚀 Docker and Aztec Node Setup in WSL (Windows Subsystem for Linux)

This guide will walk you through setting up **Docker inside WSL** to run the **Aztec node** on Windows. The setup includes configuring port forwarding, installing Docker, setting up the Aztec node, and verifying the ports are being forwarded correctly.

---

## **1️⃣ Uninstall Docker Desktop (Windows)**

Before installing Docker inside WSL, we need to uninstall **Docker Desktop** if it’s installed on your Windows system:

1. Open **Settings** > **Apps**.
2. Find **Docker Desktop**, click **Uninstall**, and follow the prompts to remove it.

---

## **2️⃣ Install Ubuntu via PowerShell**

We will install **Ubuntu** on WSL:

1. Open **PowerShell** as **Administrator**.
2. Run the following command to install **Ubuntu**:

   `wsl --install -d Ubuntu`

After installation, you can launch it from the Start Menu by searching for **Ubuntu**.

---

## **3️⃣ Enable Root Access in WSL** 🔑

By default, WSL uses a non-root user. Let’s change that:

1. **Enable root and set password**:

   Run the following command to set a password for the **root** user:

   `sudo passwd root`

2. **Set root password**: You will be prompted to enter and confirm a new password for the root user.

3. **Exit root** to return to your regular user:

   `exit`

---

## **4️⃣ Configure Port Forwarding for Aztec Node** 🔄

To forward ports from Windows to WSL, you need to use the **current IP address of your WSL instance**.

1. **Open WSL** and run the following command to get your current IP address:

   `ip addr | grep inet`

   This will output something like:
   
inet 172.18.151.151/20 brd 172.18.159.255 scope global eth0

Look for the `inet` line under **eth0** to find the **IP address** of your WSL instance. In this example, it is `172.18.151.151`.

2. **Open PowerShell as Administrator** and add port forwarding for **ports 40400** and **8080**:

Replace `172.18.151.151` with your **IP address** from the previous step.

`netsh interface portproxy add v4tov4 listenport=40400 listenaddress=0.0.0.0 connectport=40400 connectaddress=YOUR_WSL_IP`

`netsh interface portproxy add v4tov4 listenport=8080 listenaddress=0.0.0.0 connectport=8080 connectaddress=YOUR_WSL_IP`

**Example**: If your WSL IP is `172.18.151.151`, the command will look like:

`netsh interface portproxy add v4tov4 listenport=40400 listenaddress=0.0.0.0 connectport=40400 connectaddress=172.18.151.151`

`netsh interface portproxy add v4tov4 listenport=8080 listenaddress=0.0.0.0 connectport=8080 connectaddress=172.18.151.151`

---

## **5️⃣ Add Firewall Rules on Windows** 🛡️

Ensure Windows allows incoming traffic on the ports you’ve forwarded:

1. **Add firewall rules** in **PowerShell (Admin)**:

`New-NetFirewallRule -DisplayName "Allow 40400" -Direction Inbound -Protocol TCP -LocalPort 40400 -Action Allow`

`New-NetFirewallRule -DisplayName "Allow 8080" -Direction Inbound -Protocol TCP -LocalPort 8080 -Action Allow`

2. **Verify the port forwarding**:

`netsh interface portproxy show v4tov4`

You should see:

Listen on ipv4: Connect to ipv4:

0.0.0.0:40400 172.18.151.151:40400

0.0.0.0:8080 172.18.151.151:8080


---

## **6️⃣ Install Docker Inside WSL** 🐳

Now that Docker Desktop is removed, let’s install Docker inside WSL.

### **Run as root**:

`su  # Change to root user`

### **Install Docker**:

`sudo apt update -y`

`sudo apt install apt-transport-https ca-certificates curl software-properties-common -y`

`curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -`

`sudo add-apt-repository "deb [arch=amd64] https://download.docker.com/linux/ubuntu bionic stable" -y`

`sudo apt update -y`

`sudo apt install docker-ce -y`

### **Verify Docker Installation**:

`sudo docker --version`

### **Fix Docker Socket Permissions**:

`sudo chmod 777 /var/run/docker.sock`


---

## **7️⃣ Install UFW and Configure Firewall for Docker** 🔥

Ensure the firewall is correctly configured:

1. **Install UFW** (if not installed - also using root/su):

   `sudo apt install ufw -y`

2. **Allow Docker and SSH**:

   `ufw allow 22`

   `ufw allow ssh`

   `ufw enable`

3. **Allow Aztec Node Ports**:

   `ufw allow 40400`

   `ufw allow 8080`

   ### **Exit root**:

   `exit`

---

## **8️⃣ Install and Start Aztec Node** 🌐

Now, let’s install and configure your **Aztec node** on the **alpha-testnet**:

1. **Install Aztec**:

   `bash -i <(curl -s https://install.aztec.network)`

2. **Update your PATH**:

   `echo 'export PATH="$HOME/.aztec/bin:$PATH"' >> ~/.bashrc`

   `source ~/.bashrc`

3. **Start Aztec Node**:

   `aztec-up alpha-testnet`

   `aztec start --node --archiver --sequencer \`

   `--network alpha-testnet \`

   `--l1-rpc-urls RPC_URL  \`

   `--l1-consensus-host-urls BEACON_URL \`

   `--sequencer.validatorPrivateKey 0xYourPrivateKey \`

   `--sequencer.coinbase 0xYourAddress \`

   `--p2p.p2pIp YOUR_IP \`

   `--p2p.maxTxPoolSize 1000000000`

---

## **9️⃣ Confirm Ports are Being Forwarded in WSL** 🔍

While the node is running, check if ports 40400 and 8080 are being forwarded correctly (do this after 20-30mins):

`sudo lsof -iTCP -sTCP:LISTEN -P | grep -E ':8080|:40400'`

You should see something like:

docker-pr 14254 root 4u IPv4 78228 0t0 TCP *:40400 (LISTEN)

docker-pr 14261 root 4u IPv6 78233 0t0 TCP *:40400 (LISTEN)

docker-pr 14301 root 4u IPv4 78250 0t0 TCP *:8080 (LISTEN)

docker-pr 14308 root 4u IPv6 78253 0t0 TCP *:8080 (LISTEN)

---

## **Troubleshooting** 🚨

If you still encounter the **0peers** issue after completing the setup, you may need to **port forward** **ports 8080** and **40400** on your **router/modem** to ensure proper network connectivity and peer synchronization.

---

### **Credits** 🎉

- The idea of installing **Docker inside a VM** was inspired by the work of **Vikash Kumar** and his [installation script](https://github.com/vikash-kumar01/installation_scripts/blob/master/docker.sh).
- This guide adapts his method to work with **WSL** instead of a traditional VM, allowing Docker to run inside **Windows Subsystem for Linux**.
