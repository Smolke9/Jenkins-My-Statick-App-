# 🚀 HTML Static App Deployment using Jenkins & Nginx on Remote Ubuntu Server

This guide will walk you through deploying an HTML static webpage from **Jenkins** to another **Ubuntu server** using **SSH & Nginx**.

---

## 📸 Project Screenshots
- ![Step 1](Screenshot/1.jpg)
- ![Step 2](Screenshot/2.jpg)

---

## 📂 Project Structure (Local HTML Directory)

```bash
~/html$ tree
.
├── about.html
├── index.html
└── ubuntuu.pem
```

---

## 1️⃣ Prepare HTML Files & PEM Key

1. Create an HTML directory:
   ```bash
   mkdir ~/html
   cd ~/html
   ```

2. Add your HTML files:
   ```bash
   nano index.html
   nano about.html
   ```

3. Upload your `.pem` key from **local machine** to **Jenkins server**:
   ```powershell
   scp -i your-key.pem ubuntuu.pem ubuntu@<jenkins_server_ip>:~/html/
   ```

---

## 2️⃣ Push HTML Project to GitHub

1. Inside `~/html` folder:
   ```bash
   git init
   git branch -M master
   git remote add origin https://github.com/Smolke9/Jenkins-My-HTML-Static-App.git
   git add .
   git commit -m "Initial commit - HTML static site"
   git push origin master
   ```

---

## 3️⃣ Create Jenkins Freestyle Project

1. Open Jenkins → **New Item** → Select **Freestyle Project** → Name it `HTML-Static-App-Deploy`.
2. Under **Source Code Management**, select **Git**:
   - Repository URL:  
     ```
     https://github.com/Smolke9/Jenkins-My-HTML-Static-App.git
     ```
   - Branch:  
     ```
     */master
     ```

---

## 4️⃣ Set Up GitHub Webhook

1. Go to your GitHub repo → **Settings** → **Webhooks** → Add webhook:
   - Payload URL:  
     ```
     http://<jenkins_server_ip>:8080/github-webhook/
     ```
   - Content type: `application/json`
   - Select: **Just the push event**
   - Save webhook.

2. In Jenkins job → **Build Triggers** → Check:
   - `GitHub hook trigger for GITScm polling`

---

## 5️⃣ Add Jenkins Build Steps

In your Jenkins job → **Build → Execute shell**:

```bash
# Compress HTML files
zip myfile.zip ./*.html

# Secure permissions for PEM key
sudo chmod 600 ubuntuu.pem

# Transfer HTML files to remote server
scp -i ubuntuu.pem -o StrictHostKeyChecking=no myfile.zip ubuntu@3.7.55.153:.

# SSH into remote server and deploy
ssh -i ubuntuu.pem -o StrictHostKeyChecking=no ubuntu@3.7.55.153 << 'EOF'
sudo apt update
sudo apt install nginx -y
sudo apt install zip -y
sudo service nginx start

# Move uploaded zip to nginx directory
sudo cp myfile.zip /var/www/html/
cd /var/www/html

# Remove old HTML files & unzip new ones
sudo rm -f *.html
sudo unzip -o myfile.zip
EOF
```

---

## 6️⃣ Deploy & Test

- Push any changes to your GitHub repo:
  ```bash
  git add .
  git commit -m "Updated HTML"
  git push origin master
  ```
- Jenkins will automatically trigger the build & deploy to the remote server.
- Access your site at:
  ```
  http://<remote_server_ip>
  ```

---

## 📌 Notes

- Replace `3.7.55.153` with your **remote Ubuntu server's public IP**.
- Ensure `.pem` key has correct permissions (`chmod 600`).
- Open **port 80** in your remote server’s firewall or AWS Security Group.

---

✅ **You now have a fully automated HTML static site deployment from Jenkins to a remote Ubuntu server with Nginx.**
