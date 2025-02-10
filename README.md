# **FRP Server Setup on EC2**

This README provides detailed steps on how to set up and configure the **FRP server** on an **EC2 instance** running **Ubuntu**.

---

## **1. Prerequisites**

- **AWS EC2 instance** running **Ubuntu**.
- **FRP (Fast Reverse Proxy)** binary files downloaded from [GitHub Releases](https://github.com/fatedier/frp/releases).
- **Open ports:** Ensure the security group associated with the EC2 instance has the necessary ports open (e.g., port 7000 for the FRP server).
- Basic knowledge of **Linux** command line.

---

## **2. Setup FRP Server on EC2**

### **Step 1: Install Dependencies**

Ensure your EC2 instance has the necessary tools installed. You may need `wget` or `curl` to download files:

```bash
sudo apt update
sudo apt install wget curl -y
```

### **Step 2: Download and Extract FRP**

1. Download the **FRP** binary from the official GitHub releases page:

   ```bash
   wget https://github.com/fatedier/frp/releases/download/v0.47.0/frp_0.47.0_linux_amd64.tar.gz
   ```

2. Extract the tarball:

   ```bash
   tar -zxvf frp_0.47.0_linux_amd64.tar.gz
   ```

3. Navigate to the extracted folder:

   ```bash
   cd frp_0.47.0_linux_amd64
   ```

You should now have the following files:
- `frps` (FRP server binary)
- `frps.ini` (FRP server configuration file)

---

### **Step 3: Move FRP Files to a Proper Directory**

Move the `frps` binary and the `frps.ini` configuration file to a proper directory such as `/opt/frp/`:

```bash
sudo mkdir -p /opt/frp
sudo mv frps frps.ini /opt/frp/
```

---

### **Step 4: Make the FRP Binary Executable**

Ensure the `frps` binary has executable permissions:

```bash
sudo chmod +x /opt/frp/frps
```

---

## **3. Configure the FRP Server**

You will need to modify the `frps.ini` configuration file to customize the server settings. Open the `frps.ini` file:

```bash
sudo nano /opt/frp/frps.ini
```

Modify the following common settings:

- **Bind Port** (default `7000`): This is the port where the FRP server will listen for incoming connections from the client.
- **Dashboard** (optional): If you want to enable the FRP dashboard, add or modify the following settings:
```ini
[common]
bind_port = 7000          # Port for FRP to listen to
vhost_http_port = 80      # Port for HTTP requests (optional)
vhost_https_port = 443    # Port for HTTPS requests (optional)
dashboard_port = 7500     # Port for the FRP dashboard (optional)
dashboard_user = admin    # Dashboard username (optional)
dashboard_pwd = admin     # Dashboard password (optional)

```





Save and close the file (`CTRL + X`, `Y`, `ENTER`).

## **4. Start the FRP server**
Now you can start the FRP server:

```bash
   ./frps -c ./frps.ini

   ```
   
---

## **5. Create a systemd Service to Run FRP Server**

To manage the FRP server as a system service, create a systemd service file.
1. Move the extracted files to /opt/frp/ :

   ```bash
   sudo mv frp_0.47.0_linux_amd64 /opt/frp/
   ```

1. Create and edit the service file:

   ```bash
   sudo nano /etc/systemd/system/frps.service
   ```

2. Add the following content to the file:

```ini
[Unit]
Description=FRP server
After=network.target

[Service]
ExecStart=/opt/frp/frps -c /opt/frp/frps.ini
Restart=always
User=nobody
RestartSec=3

[Install]
WantedBy=multi-user.target

```

This configuration ensures that the FRP server starts on boot and restarts automatically if it crashes.

---

### **Step 5: Reload systemd and Start FRP Server**

Now, reload systemd to recognize the new service and start the FRP server:

```bash
sudo systemctl daemon-reload
sudo systemctl start frps
sudo systemctl enable frps
```

This will start the FRP server and ensure it restarts on system reboots.

---

### **Step 6: Open Necessary Ports in the Security Group**

Ensure that the required ports are open in the EC2 instance’s security group to allow traffic to the FRP server.

1. **Port 7000** (default FRP bind port): Open this for FRP to listen for client connections.
2. **Port 7500** (if using the dashboard): Open this if you want to access the FRP dashboard.

You can add rules to your security group from the AWS Management Console.

---

### **Step 7: Verify the Service Status**

Check the status of the FRP service to ensure it's running properly:

```bash
sudo systemctl status frps
```

The status should show `active (running)`.

---

## **5. (Optional) Enable FRP Dashboard**

If you've enabled the FRP dashboard (via `dashboard_port = 7500` in `frps.ini`), you can access it in your browser:

```
http://<your-ec2-public-ip>:7500
```

Login using the credentials set in `frps.ini` (default: `admin` / `admin`).

---

## **6. Connecting FRP Client**

To connect to the FRP server, you will need to configure the **FRP client** on another machine.

1. Download the client from [FRP GitHub releases](https://github.com/fatedier/frp/releases).
2. Edit the `frpc.ini` file to match the server's settings:

```ini
[common]
server_addr = <your-ec2-public-ip>
server_port = 7000

[ssh]
type = tcp
local_ip = 127.0.0.1
local_port = 22
remote_port = 6000
```

3. Start the client:

```bash
./frpc -c ./frpc.ini
```

Now, you can access the SSH service from the client by connecting to:

```bash
ssh -p 6000 user@<your-ec2-public-ip>
```

---

## **7. Troubleshooting**

If the FRP server fails to start, check the logs for more information:

```bash
sudo journalctl -u frps -xe
```

Common issues might include:
- Incorrect file paths in the service file.
- Firewall/Security group blocking the required ports.

---

## **8. Conclusion**

Your FRP server is now set up and running on your EC2 instance. You can use FRP to forward ports, access internal services, and more!

For more advanced configurations, refer to the [FRP documentation](https://github.com/fatedier/frp/blob/master/README.md).

---

Feel free to modify the README for your specific needs, but this should cover the basics of setting up and running the FRP server on an EC2 instance! Let me know if you need further assistance.
