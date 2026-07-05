


## AWS Ubuntu Setup

- Click on Launch Instance from AWS home page
- At the top name your instance something sensible that makes it clear that you're the owner of the notifies you and others of its purpose `<name-group-purpose>` is a decent naming scheme
- For the AMI select "Ubuntu Server 24.x LTS"
![AMI Selection](https://github.com/AdewuyiAdebusuyi-Sparta/flask-app-deployment/blob/main/AMI%20Selection.png)
- For instance type select t3.micro
- If you don't already have a key generated, create a new key pair, download it and keep it in a safe directory you won't forget about (a folder named .ssh at your root profile directory is a fine place)
![Network Settings](https://github.com/AdewuyiAdebusuyi-Sparta/flask-app-deployment/blob/main/Network%20Settings.png)
- Create a new security group that allows SSH traffic from Your IP (select "My IP" from the dropdown to autofill), we will add the rest of the rules later, name it according to its owner and purpose.
- You can now click launch instance
- Once your instance is running, make note of the public ipv4 address, we will use this to connect to our instance
- We also need to edit some of the security group rules to allow traffic to your instances IP, on your instance page scroll down and click security and under security groups click the link that displays the group you created earlier
- From there clock "Edit inbound rules" and change the following settings
- First make sure the SSH rule is on port 22 and set to your IP
- Add a HTTP rule set to anywhere on port 80
- Add a Custom TCP rule set to anywhere on port 3000
## Instance Dependency Setup and App Deployment

Once we've setup the instance we'll need to run git bash to access the instance. Before we enter into the machine we need to check if the app we need to transfer to the instance is available to download through GitHub or some other service, or that we need to transfer the files from our local machine to the instance.

to transfer files locally we need to use the  ```scp``` command to transfer the them.
the full command in this case is:
- ```scp -i <SSH key file location> <local_file_path> <username>@<ubuntu_public_ip>:<destination path>```
The  ```-i``` flag is used to specify the SSH key you're using, this is also used to login to the instance, in the case of creating ubuntu instances with AWS, the username will be "ubuntu"
at the very least we'll need to transfer our API key file and copy it to our instance 

we can now enter the machine with the ```ssh``` command:
the full command in this case is:
- ```ssh -i <SSH_key_file_location> <username>@<ubuntu_public_ip```

from there we will have entered the machine and will need to input the following commands
```bash
#update the packages on the machine, you may add the -y flag to any of the installations to automatically say yes to any prompts
sudo apt update
sudo apt upgrade

sudo apt install nginx
```

If we're cloning a repo to access the app follow this code block
```bash
#the name of the folder you'll need to cd into will match the repo name
git clone <repo_github_url>
mv ~/api_key <repo_name>
cd <repo_name>
```
otherwise
```bash
#we need to unzip the app since we transferred it as a zip through scp
sudo apt install unzip
unzip <app_file>
#if we didn't include the api_key in the zip
mv ~/api_key <repo_name>
cd app
```
now continuing:



```bash
#searches for the first string and replaces it with the second, we're rerouting localhost:3000 to the public IP
sudo sed -i 's|try_files \$uri \$uri/ =404;|proxy_pass http://localhost:3000;|' /etc/nginx/sites-available/default
#restarts nginx so our changes take place
systemctl restart nginx
```

```bash
python3 <filename>.py
```

We should now have a working API accessible by anyone as long as they're given the public IP and are aware of the calls that they can make
