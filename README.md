# AWS EC2 Deployment of Node.js Application

---

## Steps

### 1. Project Setup

1. **Clone the Project**:
   - Create a `.env` file and add the following environment variables:
     ```env
     DOMAIN="http://localhost:3000"
     PORT=3000
     STATIC_DIR="./client"

     # Stripe API keys
     PUBLISHABLE_KEY="YOUR_PUBLISHABLE_KEY"
     SECRET_KEY="YOUR_SECRET_KEY"
     ```
   - **Note**: Obtain the Stripe API keys from your [Stripe account](https://stripe.com).

2. **Run Locally**:
   - Run the app locally to test if it’s working before deploying.

---

### 2. AWS EC2 Setup

1. **Create an IAM User**:
   - In AWS, create an IAM user with administrative access.
   - Log in as the IAM user to create and manage your EC2 instance.

2. **Create an EC2 Instance**:
   - Choose **Ubuntu** as the OS, select the region, configure a VPC, and create a key pair (`.pem` file).
   - Launch the EC2 instance.

3. **Configure Key Permissions on Local Machine**:
   - **For PowerShell**: Open PowerShell in the folder containing your `.pem` file and set permissions (since `chmod` doesn’t work in PowerShell):
     ```powershell
     icacls "yourkey.pem" /inheritance:r
     icacls "yourkey.pem" /grant:r "%username%:R"
     icacls "yourkey.pem" /remove "NT AUTHORITY\Authenticated Users"
     icacls "yourkey.pem" /remove "BUILTIN\Users"
     ```
   - **For Linux**:
     ```bash
     chmod 400 yourkey.pem
     ```

4. **Connect to EC2 Instance**:
   - Connect via SSH:
     ```bash
     ssh -i "yourkey.pem" ubuntu@<EC2-Public-DNS>
     ```

5. **Install Dependencies**:
   - Update and install necessary packages on the instance:
     ```bash
     sudo apt update
     sudo apt install -y git nodejs npm
     ```

6. **Clone GitHub Repository**:
   - Clone your repository and navigate to the project folder:
     ```bash
     git clone https://github.com/your-username/aws-ec2-nodejs-app.git
     cd aws-ec2-nodejs-app
     ```

7. **Install Node Modules**:
   - Install project dependencies:
     ```bash
     npm install
     ```

8. **Set Up `.env` on EC2**:
   - Create and edit the `.env` file directly on the EC2 instance:
     ```bash
     touch .env
     vi .env  # Add environment variables as needed
     ```

9. **Start the Application**:
   - Start the Node.js app:
     ```bash
     npm start
     ```

10. **Configure EC2 Security Group**:
    - In the AWS Console, go to **Security Group** for your instance.
    - Allow traffic on port `3000` by adding a new rule for **Custom TCP**, port `3000`, accessible from **Anywhere (IPv4)**.

11. **Access the App**:
    - Use the EC2 public IP with port `3000` to view your app: `http://<EC2-Public-IP>:3000`.

---

### 3. GitHub Actions for Continuous Deployment

1. **Add Secrets**:
   - In your GitHub repository, go to **Settings > Secrets and Variables > Actions** and add the following secrets:
     - `EC2_HOST`: Public DNS or IP of your EC2 instance (e.g., `ec2-100-26-43-234.compute-1.amazonaws.com`).
     - `EC2_USER`: `ubuntu` (the default user for Ubuntu on EC2).
     - `EC2_KEY`: The content of your `.pem` key.

2. **GitHub Actions Workflow File**:
   - In your project, create `.github/workflows/deploy.yml` with the following content:
     ```yaml
     name: Deploy to EC2

     on:
       push:
         branches:
           - main

     jobs:
       deploy:
         runs-on: ubuntu-latest

         steps:
           - name: Checkout code
             uses: actions/checkout@v2

           - name: Add SSH key
             env:
               EC2_KEY: ${{ secrets.EC2_KEY }}
             run: |
               mkdir -p ~/.ssh
               echo "${EC2_KEY}" > ~/.ssh/ec2-key.pem
               chmod 600 ~/.ssh/ec2-key.pem

           - name: Deploy to EC2
             env:
               HOST: ${{ secrets.EC2_HOST }}
               USER: ${{ secrets.EC2_USER }}
             run: |
               ssh -o StrictHostKeyChecking=no -i ~/.ssh/ec2-key.pem $USER@$HOST << 'EOF'
                 cd /home/ubuntu/aws-ec2-nodejs-app
                 git pull origin main
                 npm install
                 pm2 restart your-app-name || pm2 start app.js --name your-app-name
               EOF
     ```
   - This workflow automatically deploys your app to EC2 each time you push changes to the `main` branch.

---

## Notes

- Make sure `pm2` is installed on your EC2 instance for process management.
- Replace `your-app-name` with the desired name for your app when using `pm2`.
- Verify environment variables and secrets before running the deployment workflow.

--- 

This completes the setup for deploying a Node.js app to AWS EC2 with GitHub Actions for CI/CD. Enjoy your deployment!
