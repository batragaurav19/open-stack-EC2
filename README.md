# **Deploying OpenStack on an EC2 Instance**

### **💻 Local Machine Setup**
- **Windows:** PuTTY + PuTTYgen  
- **Linux/macOS:** Built-in terminal (Just `ssh` away!)  

---

## **🔌 Step 1: Connecting to the EC2 Instance**
Time to jump into your cloud machine!  

### **For Windows (PuTTY Users)**
```markdown
1. Convert `.pem` to `.ppk` using PuTTYgen  
2. Open PuTTY → Enter:  
   - Host: `EC2_Public_IP`  
   - Load your `.ppk` key  
3. Login as `ubuntu`  
```

### **For Linux/macOS (Terminal Pros)**
```bash
chmod 400 your-key.pem  # Lock it down!
ssh -i your-key.pem ubuntu@EC2_Public_IP
```
**💡 Pro Tip:** Use `tmux` or `screen` to avoid disconnections!  

---

## **⚙️ Step 2: Update & Install Dependencies**
Let’s get the system ready for OpenStack magic!  

```bash
sudo apt update && sudo apt upgrade -y  # Fresh & updated!
sudo apt install -y git python3-dev python3-pip net-tools  # Essentials!
```

---

## **📥 Step 3: Clone DevStack Repository**
DevStack = Fastest way to deploy OpenStack!  

```bash
git clone https://opendev.org/openstack/devstack.git
cd devstack  # Let’s get into the action!
```

![DevStack Clone](https://github.com/user-attachments/assets/a832551c-d86d-4a7c-9e32-6998b75c7846)

---

## **🔧 Step 4: Configure DevStack**
Let’s customize OpenStack before installation!  

```bash
nano local.conf  # Time for some config magic!
```
Paste this **optimized** config:  
```ini
[[local|localrc]]
ADMIN_BATRA=SuperSecret
DATABASE_PASSWORD=$ADMIN_BATRA
RABBIT_PASSWORD=$ADMIN_BATRA
SERVICE_PASSWORD=$ADMIN_BATRA

# Set Host IP (Detects Automatically)
HOST_IP=$(hostname -I | awk '{print $1}')

# Enable Only Essential Services
disable_service tempest
disable_service cinder
disable_service swift
disable_service heat

# Use Latest OpenStack Release
USE_PYTHON3=True
GIT_BASE=https://opendev.org
```

## **🚀 Step 5: Install OpenStack!**
Hold tight—this will take **30-45 mins**! ☕  

```bash
./stack.sh  # Let the magic begin!
```

![OpenStack Installation](https://github.com/user-attachments/assets/a0a40b44-e7d3-4005-b076-756a8e9281ee)

---

## **🖥️ Step 6: Access Horizon Dashboard**
Your OpenStack GUI is ready!  

🌐 **URL:** `http://EC2_Public_IP/dashboard`  

![ec9 (1)](https://github.com/user-attachments/assets/30f24196-0a83-42cd-a758-c8867cb47a79)

🔑 **Login:**  
- **Username:** `admin`  
- **Password:** `SuperSecret`  

![ec10 (1)](https://github.com/user-attachments/assets/1991edd9-ba34-41cc-a6ec-ecc130b9105e)

---

## **✅ Step 7: Verify Services**
Let’s make sure everything’s running smoothly!  

```bash
source openrc admin admin  # Load OpenStack credentials
openstack service list  # Check all services
```

### **🔍 Quick Service Checks**
| Service  | Command |
|----------|---------|
| **Keystone** | `openstack token issue` |
| **Nova** | `openstack compute service list` |
| **Neutron** | `openstack network agent list` |
| **Glance** | `openstack image list` |

![image](https://github.com/user-attachments/assets/bc6c8902-e8bc-453a-8848-14564446f381)

![image](https://github.com/user-attachments/assets/7ef9c56a-4ca8-4df2-b382-767b60f65dcc)

![image](https://github.com/user-attachments/assets/fac9a2c4-7734-4f1a-86e2-62c134af4faf)
---

## **🛠️ Step 8: Troubleshooting**
Hit a snag? Let’s fix it!  

### **Common Issues & Fixes**
| Problem | Solution |
|---------|----------|
| **Horizon not loading** | `sudo systemctl restart apache2` |
| **Nova service down** | `sudo systemctl restart devstack@n-api` |
| **Networking issues** | Check `/var/log/neutron/*.log` |

### **📂 Log Locations**
- **DevStack logs:** `/opt/stack/logs`  
- **Service logs:** `/var/log/[service_name]`  

---

# **🖥️ Step 9: Deploying a VM in OpenStack Horizon** 🎮
Time to launch your **first cloud VM**! Follow these steps carefully.  

---

## **🔑 Step 1: Create a Key Pair**
You’ll need this to SSH into your VM!  

1. Go to **Project → Compute → Key Pairs**  
2. Click **Create Key Pair**  
   - **Name:** `test-key`  
   - **Key Type:** `SSH Key (RSA)`  
3. **Download** the `.pem` file (Keep it safe!)  

![ec14 (1)](https://github.com/user-attachments/assets/6187e32a-adbe-438f-b947-b6fe91a28201)

---

## **🖼️ Step 2: Upload a Cloud Image**
We’ll use **Ubuntu 22.04** for the VM.  

1. Navigate to **Project → Compute → Images**  
2. Click **Create Image**  
   - **Name:** `Ubuntu-22.04`  
   - **Image Source:** `URL`  
   - **URL:**  
     ```bash
     https://cloud-images.ubuntu.com/jammy/current/jammy-server-cloudimg-amd64.img
     ```
3. Wait for **Active** status  
![ec15 (1)](https://github.com/user-attachments/assets/59224680-0685-45ef-8641-a39076c0fcda)



---

## **⚡ Step 3: Define a Flavor**
A **flavor** = VM size (CPU, RAM, Disk).  

1. Go to **Admin → Compute → Flavors**  
2. Click **Create Flavor**  
   - **Name:** `small`  
   - **vCPUs:** `1`  
   - **RAM:** `2048 MB`  
   - **Disk:** `10 GB`  

![ec16 (1)](https://github.com/user-attachments/assets/c1ddad08-83c2-4459-9464-b2d4268905df)



---

## **🌐 Step 4: Set Up Networking**
A VM needs a network!  

### **A. Create a Private Network**
1. **Project → Network → Networks**  
2. **Create Network**  
   - **Name:** `private-net`
   - **Subnet Name:** `private-subnet`
   - **Subnet Address:** `192.168.1.0/24`  

### **B. Add a Router**
1. **Project → Network → Routers**  
2. **Create Router** → Name: `my-router`

Now `delete` the default `router` and go back to `Networks` and delete the `private network` as well.

![ec17 (1)](https://github.com/user-attachments/assets/351faba4-56eb-4101-a00d-e0194f913bf7)



---

## **🚀 Step 5: Launch the VM!**
Time for liftoff!  

1. Go to **Project → Compute → Instances**  
2. Click **Launch Instance**  
   - **Name:** `my-instance`  
   - **Flavor:** `small`  
   - **Image:** `Ubuntu-22.04`  
   - **Network:** `private-net`  
   - **Key Pair:** `test-key`  

![ec18 (1)](https://github.com/user-attachments/assets/53f06a42-6497-46e7-82a7-c492ee5747b5)



---

## **🔐 Step 6: SSH into Your VM**
Once the VM is **Active**, connect via:  

```bash
chmod 400 test-key.pem
ssh -i my-key.pem ubuntu@<VM_IP>
```
**🎉 Congrats! You’re in your OpenStack VM!**  

---

## **🎉 Conclusion**
You’ve just **built your own private cloud** on AWS! 🎉  

### **What’s Next?**
- **Scale up:** Add more nodes!  
- **Automate:** Use Terraform/Ansible!  
- **Experiment:** Try Kubernetes on OpenStack!  

**Happy Cloud Building!** ☁️🚀  

**References:**
- [DevStack Documentation](https://docs.openstack.org/devstack/latest/)
- [OpenStack CLI Reference](https://docs.openstack.org/python-openstackclient/latest/cli/)
---

**💬 Got questions?**  
Drop them below! Let’s make cloud computing **fun and easy**! 😃
