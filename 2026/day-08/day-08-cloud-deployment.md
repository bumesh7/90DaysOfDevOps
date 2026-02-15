Part 1: Launch Cloud Instance & SSH Access (15 minutes)
Step 1: Create a Cloud Instance

Step 2: Connect via SSH

Part 2: Install Docker & Nginx (20 minutes)
Step 1: Update System
sudo apt-get update

Step 3: Install Nginx

sudo apt install nginx
sudo syatemctal start nginx
sudo systemctl status nginx

Verify Nginx is running:

<img width="1466" height="358" alt="image" src="https://github.com/user-attachments/assets/05eb3882-7119-4b7a-8db4-3f673aa8ab5f" />


Part 3: Security Group Configuration (10 minutes)
Test Web Access: Open browser and visit: http://<your-instance-ip>

You should see the Nginx welcome page!

📸 Screenshot this page - you'll need it for submission

<img width="1565" height="237" alt="image" src="https://github.com/user-attachments/assets/e04931d5-3548-4d29-ba8f-24b26c13cff2" />

<img width="1383" height="381" alt="image" src="https://github.com/user-attachments/assets/6d3e0cb8-6b10-443b-b8d4-d98b8b4fed34" />



Part 4: Extract Nginx Logs (15 minutes)
Step 1: View Nginx Logs
<img width="1912" height="185" alt="image" src="https://github.com/user-attachments/assets/a9b610fb-067d-4359-a868-c9894b5bc590" />


Step 2: Save Logs to File

Step 3: Download Log File to Your Local Machine

# On your local machine (new terminal window)
# For AWS:
scp -i your-key.pem ubuntu@<your-instance-ip>:~/nginx-logs.txt .

example: scp -i ShellScript.pem ubuntu@ec2-3-106-139-34.ap-southeast-2.compute.amazonaws.com:~/nginx_logs.txt .
               

# For Utho:
scp root@<your-instance-ip>:~/nginx-logs.txt .
