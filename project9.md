***Step 1 – Install Jenkins server***

1. Create an AWS EC2 server based on Ubuntu Server 20.04 LTS and name it "Jenkins"
![](images/1.PNG)

2. Install JDK
``` 
sudo apt update    
sudo apt install default-jdk-headless 
```
![](images/2.PNG)

3. Install Jenkins
``` 
wget -q -O - https://pkg.jenkins.io/debian-stable/jenkins.io.key | sudo apt-key add -
sudo sh -c 'echo deb https://pkg.jenkins.io/debian-stable binary/ > \
    /etc/apt/sources.list.d/jenkins.list'
sudo apt update
sudo apt-get install jenkins 
```
![](images/3.PNG)

Check if Jenkins is up and running
```
sudo systemctl status jenkins
```
![](images/4.PNG)

4. Open port 8080 for Jenkins server by creating a new Inbound Rule in EC2 Security Group
![](images/5.PNG)

5. Perform initial Jenkins setup
![](images/6.PNG)
![](images/7.PNG)
![](images/8.PNG)

***Step 2 – Configure Jenkins to retrieve source codes from GitHub using Webhooks***

1. Enable webhooks in GitHub repository settings
![](images/10.PNG)

2. Create a new item using a Freestyle project on Jenkins web console
![](images/11.PNG)

To connect to my GitHub repository, I copied the tooling GitHub repository HTTPS url code and paste it in Jenkins project9 source code management, created a credentials so that Jenkins could access files in the repository then saved the configuration 
![](images/13.PNG)

Click "Build Now" button, if everything has been configured correctly, the build will be successfull and you will see it under # green hash symbol and the number 
![](images/14.PNG)

Open the build and check in "Console Output" if it has run successfully.
![](images/15.PNG)


3. Click "Configure" your job/project and add these two configurations

- Configure triggering the job from GitHub webhook 
![](images/16.PNG)
- Configure "Post-build Actions" to archive all the files – files resulted from a build are called "artifacts".
![](images/17.PNG)

I edited the READ.MD file in my tooling github repository and added a sentence and clicked on coomit changes
![](images/18.PNG)

You will see that a new build has been launched automatically (by webhook) and you can see its results – artifacts, saved on Jenkins server.
![](images/19.PNG)

By default, the artifacts are stored on Jenkins server locally

```
ls /var/lib/jenkins/jobs/project9/builds/4/archive/ 
```
![](images/20.PNG)

***Step 3 – Configure Jenkins to copy files to NFS server via SSH***

1. Install "Publish Over SSH" plugin.
![](images/21.PNG)

2. Configure the job/project to copy artifacts over to NFS server.

On the main dashboard select "Manage Jenkins" and click on "Configure System" then add the following configuration
- Provide a private key (content of .pem file that you use to connect to NFS server via SSH/Putty)
Arbitrary name
- Hostname – IP address of the NFS server
- Username – ec2-user (since NFS server is based on EC2 with RHEL 8)
- Remote directory – /mnt/apps since our Web Servers use it as a mointing point to retrieve files from the NFS server
![](images/22.PNG)

Open Jenkins job/project configuration page and add another one "Post-build Action"
Select "Send build artifacts over SSH" and configure it to send all files probuced by the build into our previouslys define remote directory. I have used ** because I want to copy all files and directories
![](images/24.PNG)

I changed something in README.MD file in my GitHub Tooling repository which triggers the webhook new job and in the "Console Output" of the job, it displays the following output on the screenshot
![](images/25.PNG)

To make sure that the files in /mnt/apps have been udated – connect via SSH/Putty to your NFS server and check README.MD file

```
cat /mnt/apps/README.md
```
![](images/26.PNG)






