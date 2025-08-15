# Configuration-Management-with-Ansible

The listed projects in this module cover:

---

### **1. First Playbooks**
- **Playbook anatomy**: plays, tasks, modules, YAML syntax.  
- **Demo**:  
  - Install `nodejs`, `npm`  
  - Deploy a **Node.js app** via **git clone + pm2**  
  - Restart service with `systemd` module.

---

### **2. Automate Nexus Repository**
- **Demo**:  
  - Install Java, download & unpack Nexus.  
  - Create dedicated `nexus` user, systemd service, and expose port `8081`.

---

### **3. Ansible + Docker**
- **Provision EC2** with Terraform → **hand over to Ansible**.  
- **Plays**: install Docker, docker-compose, add user to `docker` group.  
- **Pull & start multi-container stack** (Java + MySQL) from `docker-compose.yml`.

---

### **4. Dynamic Inventory**
- Replaced static `hosts` file with **AWS EC2 inventory plugin**.  
- **Auto-discovers** running EC2s by tag, key or subnet; no manual IP lists.

---

### **5. Kubernetes Automation**
- Terraform creates **EKS cluster**.  
- Ansible uses **k8s module** to:  
  - Create **namespace**  
  - Deploy **manifests** (Deployment + Service)  
  - Set **kubeconfig** via env var.

---

### **6. Jenkins-Driven Ansible**
- **Dedicated Ansible control node**.  
- **Jenkins pipeline** stages:  
  1. Checkout repo  
  2. Install Ansible plugin  
  3. Run playbook(s) via **SSH agent**  
  4. Output playbook results back to Jenkins console.

---

### **7. Refactor with Roles**
- Converted monolithic playbooks into **Ansible Roles** (`tasks/`, `vars/`, `templates/`).  
- **Re-used roles** (`docker`, `nodejs`, `nexus`) across **Terraform-created EC2s**.
