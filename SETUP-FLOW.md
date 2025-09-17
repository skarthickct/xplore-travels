# Complete Setup Flow

## 🚀 **Step-by-Step Deployment Guide**

### **Step 1: Create EC2 Instance**

1. **Launch EC2 Instance**
   - AMI: Ubuntu 22.04 LTS
   - Instance Type: t2.micro (free tier)
   - Security Group: Allow SSH (22) and HTTP (5000)

2. **Configure Security Group**
   ```
   Type: SSH, Port: 22, Source: Your IP
   Type: Custom TCP, Port: 5000, Source: 0.0.0.0/0
   ```

3. **Create/Download Key Pair**
   - Save `your-key.pem` file securely
   - `chmod 600 your-key.pem`

### **Step 2: Setup EC2 with User Data**

**Option A: User Data Script (Launch time)**
```bash
#!/bin/bash
apt update
apt install -y docker.io
systemctl start docker
systemctl enable docker
usermod -aG docker ubuntu
```

**Option B: Manual Setup (After launch)**
```bash
# SSH to EC2
ssh -i your-key.pem ubuntu@YOUR_EC2_IP

# Install Docker
sudo apt update
sudo apt install -y docker.io
sudo systemctl start docker
sudo systemctl enable docker
sudo usermod -aG docker ubuntu

# Logout and login again
exit
ssh -i your-key.pem ubuntu@YOUR_EC2_IP
```

### **Step 3: Create Docker Hub Account**

1. Go to https://hub.docker.com
2. Create account or login
3. Create repository: `your-username/xplore-travels`
4. Generate Access Token:
   - Account Settings → Security → New Access Token
   - Save token securely

### **Step 4: Create GitHub Repository**

1. **Create New Repository**
   - Name: `xplore-travels-website`
   - Visibility: Public/Private
   - Initialize with README: No

2. **Push Local Code**
   ```bash
   cd /mnt/d/Xplore-Travels-website
   git init
   git add .
   git commit -m "Initial commit"
   git branch -M main
   git remote add origin https://github.com/YOUR_USERNAME/xplore-travels-website.git
   git push -u origin main
   ```

### **Step 5: Configure GitHub Secrets**

1. **Go to Repository Settings**
   - Settings → Secrets and variables → Actions

2. **Add Required Secrets**
   ```
   DOCKER_USERNAME = your_dockerhub_username
   DOCKER_PASSWORD = your_dockerhub_access_token
   EC2_SSH_KEY = contents_of_your_key.pem_file
   EC2_HOST = your_ec2_public_ip
   ```

3. **Get SSH Key Content**
   ```bash
   cat your-key.pem
   # Copy entire content including -----BEGIN/END----- lines
   ```

### **Step 6: Test Manual Deployment**

1. **SSH to EC2**
   ```bash
   ssh -i your-key.pem ubuntu@YOUR_EC2_IP
   ```

2. **Test Docker**
   ```bash
   docker --version
   docker run hello-world
   ```

3. **Manual Deploy Test**
   ```bash
   # Pull and run (replace YOUR_USERNAME)
   docker pull YOUR_USERNAME/xplore-travels:latest
   docker run -d --name xplore-travels -p 5000:5000 YOUR_USERNAME/xplore-travels:latest
   
   # Test access
   curl http://localhost:5000
   ```

### **Step 7: Trigger GitHub Actions**

1. **Make a Change**
   ```bash
   # Edit any file in app/ directory
   echo "# Updated" >> app/README.md
   git add .
   git commit -m "Trigger deployment"
   git push origin main
   ```

2. **Monitor Workflow**
   - Go to GitHub → Actions tab
   - Watch workflow execution
   - Check for any errors

### **Step 8: Verify Deployment**

1. **Check Application**
   ```
   http://YOUR_EC2_IP:5000
   ```

2. **SSH and Verify**
   ```bash
   ssh -i your-key.pem ubuntu@YOUR_EC2_IP
   docker ps | grep xplore-travels
   docker logs xplore-travels
   ```

## 📋 **Quick Reference Checklist**

- [ ] EC2 instance created with security groups
- [ ] Docker installed on EC2
- [ ] Docker Hub account and repository created
- [ ] GitHub repository created and code pushed
- [ ] GitHub secrets configured (4 secrets)
- [ ] Manual deployment tested
- [ ] GitHub Actions workflow triggered
- [ ] Application accessible on EC2 IP:5000

## 🔧 **Common Issues & Solutions**

**EC2 Connection Issues**
```bash
# Check security group allows SSH from your IP
# Verify key file permissions: chmod 600 your-key.pem
```

**Docker Permission Denied**
```bash
# Add user to docker group and re-login
sudo usermod -aG docker ubuntu
exit && ssh -i your-key.pem ubuntu@YOUR_EC2_IP
```

**GitHub Actions Fails**
```bash
# Check secrets are correctly set
# Verify EC2 is accessible from GitHub (security group)
# Ensure Docker Hub credentials are valid
```

**Application Not Accessible**
```bash
# Check security group allows port 5000
# Verify container is running: docker ps
# Check logs: docker logs xplore-travels
```

## 🎯 **Success Criteria**

✅ EC2 instance running with Docker  
✅ GitHub repository with workflow  
✅ Secrets configured correctly  
✅ Application deploys automatically  
✅ Website accessible at http://EC2_IP:5000  

## 📞 **Next Steps**

1. **Custom Domain**: Point domain to EC2 IP
2. **SSL Certificate**: Add HTTPS with Let's Encrypt
3. **Load Balancer**: Use ALB for production
4. **Monitoring**: Add CloudWatch or Prometheus
5. **Backup**: Database backup strategy
