**Complete Step-by-Step Pi-hole Setup on Raspberry Pi 3 with Docker**

This guide documents every step required to set up Pi-hole inside Docker on a Raspberry Pi 3 running Raspberry Pi OS Lite (32-bit). It includes all necessary configurations, commands, and considerations.

---

## **Step 1: Install Raspberry Pi OS Lite on the SD Card**

### **1. Download Raspberry Pi OS Lite (32-bit)**
- Obtain the image from the official Raspberry Pi website.
- Choose **"Raspberry Pi OS Lite (Legacy, 32-bit)"** for best compatibility with the Raspberry Pi 3.

### **2. Flash the OS to an SD Card**
- Use **Raspberry Pi Imager** to write the image to a microSD card.
- Select **Raspberry Pi OS Lite (Legacy, 32-bit)** as the operating system.
- Choose the correct SD card.

### **3. Configure OS Customization Settings (Before Writing the Image)**
- **Set Hostname:** `pihole.local`
- **Set Username & Password:**
- Username: `user`
- Password: `[entered password]`
- **Enable SSH:**
- Allowed **public-key authentication only**.
- Copied SSH public key for secure authentication.
- **Set Locale & Keyboard Settings:**
- Time zone: `America/Chicago`
- Keyboard layout: `US`
- **Checked** "Eject media when finished".
- **Did NOT configure Wi-Fi** (used **Ethernet** for stability).

### **4. Flash OS to SD Card and Boot Raspberry Pi**
- Insert the microSD card into the **Raspberry Pi 3**.
- Connect the Raspberry Pi to power and boot it up.
- Use a **wired Ethernet connection** for stability.

---

## **Step 2: Connect to the Raspberry Pi via SSH**

### **1. Find the Raspberry Pi’s IP Address**
- Run the following command from a local machine:
```sh
ping pihole.local
```
- Alternatively, check the device’s IP address in the **router’s admin panel**.

### **2. Connect via SSH**
- Use the following command to SSH into the Raspberry Pi:
```sh
ssh user@pihole.local
```
- Confirm successful SSH connection.

---

## **Step 3: Update and Prepare the System**

### **1. Update the System**
- Ensure the latest security updates and package versions are installed:
```sh
sudo apt update && sudo apt upgrade -y
```

### **2. Install Required Dependencies**
- Install necessary packages:
```sh
sudo apt install -y curl git
```
- `curl` is used for downloading external scripts.
- `git` is useful for future installations (not required for Pi-hole).

---

## **Step 4: Install Docker**

### **1. Install Docker Using the Official Script**
- Run the following command:
```sh
curl -fsSL https://get.docker.com | sh
```
- This installs the latest version of **Docker**.

### **2. Add the User to the Docker Group**
- Enable running Docker without `sudo`:
```sh
sudo usermod -aG docker $USER
```

### **3. Apply Group Changes Without Rebooting**
```sh
newgrp docker
```

### **4. Enable and Start Docker Service**
```sh
sudo systemctl enable docker
sudo systemctl start docker
```

### **5. Verify Docker Installation**
```sh
docker --version
docker ps
```
- Output confirms that Docker is successfully installed.

---

## **Step 5: Install Docker Compose**

### **1. Install Docker Compose**
```sh
sudo apt install -y docker-compose
```

### **2. Verify Installation**
```sh
docker-compose --version
```
- Output confirms `docker-compose` was installed (v1.29.2).

---

## **Step 6: Create the docker-compose.yml File**

### **1. Create Pi-hole Directory and Config File**
```sh
mkdir -p ~/pihole && cd ~/pihole
nano docker-compose.yml
```

### **2. Add the Following Configuration**
```yaml
version: '3'
services:
pihole:
container_name: pihole
image: pihole/pihole:latest
restart: unless-stopped
network_mode: "host"
environment:
TZ: "America/Chicago"
WEBPASSWORD: "yourpassword"
DNSMASQ_LISTENING: "all"
PIHOLE_DNS_: "8.8.8.8;8.8.4.4"
volumes:
- './pihole/etc-pihole:/etc/pihole'
- './pihole/etc-dnsmasq:/etc/dnsmasq.d'
cap_add:
- NET_ADMIN
```
- Save and exit: **CTRL + X, then Y, then ENTER**.

---

## **Step 7: Start Pi-hole in Docker**

### **1. Start Pi-hole Container**
```sh
docker-compose up -d
```

### **2. Verify Container is Running**
```sh
docker ps
```

---

## **Step 8: Configure Network to Use Pi-hole**

### **1. Find Raspberry Pi’s IP Address**
```sh
hostname -I
```
- Example output: `192.168.1.2 172.17.0.1`
- Use `192.168.1.2` as the primary DNS server.

### **2. Configure Asus Router DNS Settings**
- **WAN Settings**:
- Go to **WAN** → **Internet Connection**.
- Under **WAN DNS settings**, set the **DNS server** to `192.168.1.2`.
- **LAN Settings**:
- Navigate to **LAN** → **DHCP Server**.
- Change **DNS Server 1** to `192.168.1.2`.
- Set **Advertise Router’s IP in addition to User Specified DNS** to **No**.
- Enable **Manual Assignment** and assign `192.168.1.2` to the Pi-hole device.
- **Restart the router** to apply the changes and ensure the new DNS settings take effect.

---

## **Step 9: Access Pi-hole Web Interface**

### **1. Open the Web Interface**
- Navigate to: `http://192.168.1.2/admin`
- Log in with the **password** set in `docker-compose.yml`.

### **2. Adjust Privacy Settings**
- **Go to** `Settings` → `Privacy`
- Change **DNS resolver privacy level** to `High (Domains Display & Store All Domains is Hidden)`.

### **3. Adjust DNS Settings**
- **Go to** `Settings` → `DNS`
- Change **Interface Settings** to **Allow Only Local Requests**.

---

## **Pi-hole Setup Successfully Completed**
Pi-hole is fully operational inside Docker on a Raspberry Pi 3.
