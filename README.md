????https://dev.to/yeshwanthlm/deploy-resume-to-aws-ec2-using-github-actions-4hog???????

Host Your Resume on AWS EC2 with a CI/CD Setup Using GitHub Actions
This article helps you understand how you can automatically deploy your code to AWS EC2 from GitHub

Step1: Create an EC2 Instance and Download the Key Pair.

Step2: Create Secrets in GitHub for the Repository

Step3: Creating your first workflow

Step4: Testing

Creating your first workflow
Create a .github/workflows directory in your repository on GitHub if this directory does not already exist.
In the .github/workflows directory, create a file named github-actions-ec2.yml.
Now your github-actions-ec2.yml should be present in .github/workflows/github-actions-ec2.yml in your repository

Start your file by defining jobs, jobs are the steps that you can define and see individual status reports when you see the logs in your Actions tab

jobs:
  deploy:
    name: Deploy to EC2
    runs-on: ubuntu-latest
In the above block we have defined our job with name Deploy to EC2 and enforced it to run on latest Ubuntu by runs-on: ubuntu-latest line

Now, we need to checkout the pushed code to the runner by using a predefined action named actions/checkout@v2. The code responsible for this step should look like the following

steps:
  - name: Checkout the files
    uses: actions/checkout@v2
Now, we are deploying the code to the server, in order to to do this we need to access the EC2 using ssh and perform rsync form the runner. For this we are going to use another GitHub action easingthemes/ssh-deploy

- name: Deploy to Server 1
  uses: easingthemes/ssh-deploy@main
  env:
    SSH_PRIVATE_KEY: ${ { secrets.EC2_SSH_KEY }}
    REMOTE_HOST: ${ { secrets.HOST_DNS }}
    REMOTE_USER: ${ { secrets.USERNAME }}
    TARGET: ${ { secrets.TARGET_DIR }}
Note: You need to put the double parentheses together; I had to leave a space because my code formatter refuses to print it (:facepalm)

You need to fill in the secrets using GitHub Secrets that you can add in your repo, read GitHub Secrets

EC2_SSH_KEY: This will be your .pem file which you will use to login to the instance

HOST_DNS: Public DNS record of the instance, it will look something like this ec2-xx-xxx-xxx-xxx.us-west-2.compute.amazonaws.com

USERNAME: Will be the username of the EC2 instance, usually ubuntu

TARGET_DIR: Is where you want to deploy your code.
Once you add all these information your repo will look like thisGitHub Secrets

Image description

Trigger deployment only on push to master branch

Add the following code so that your actions only run when you push to main branch.

on:
  push:
    branches:
      - main
The final .github/workflows/github-actions-ec2.yml should looks like the following

name: Push-to-EC2

# Trigger deployment only on push to main branch
on:
  push:
    branches:
      - main

jobs:
  deploy:
    name: Deploy to EC2 on master branch push
    runs-on: ubuntu-latest

    steps:
      - name: Checkout the files
        uses: actions/checkout@v2

      - name: Deploy to Server 1
        uses: easingthemes/ssh-deploy@main
        env:
          SSH_PRIVATE_KEY: ${{ secrets.EC2_SSH_KEY }}
          REMOTE_HOST: ${{ secrets.HOST_DNS }}
          REMOTE_USER: ${{ secrets.USERNAME }}
          TARGET: ${{ secrets.TARGET_DIR }}

      - name: Executing remote ssh commands using ssh key
        uses: appleboy/ssh-action@master
        with:
          host: ${{ secrets.HOST_DNS }}
          username: ${{ secrets.USERNAME }}
          key: ${{ secrets.EC2_SSH_KEY }}
          script: |
            sudo apt-get -y update
            sudo apt-get install -y apache2
            sudo systemctl start apache2
            sudo systemctl enable apache2
            cd home
            sudo mv * /var/www/html


Redis is #1 most used for AI agent data storage and NoSQL databases. Here's why.
49,000 responses in 
