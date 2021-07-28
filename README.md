# 2tiera app deployment
## Setting up new AWS ec2 instance.

- Make sure to be logged in and have location set to ireland

![Capture3](https://user-images.githubusercontent.com/86292184/127200568-a6006f11-5ad3-42be-ad1a-e24a72e97f06.PNG)


<br> </br>
- ------------------------------------
<br> </br>
**SSH 	TIMEOUT ERROR FIX**
- Likely when you restart your instace, aws allocates new IP.
-To reset IP: 
	- Open EC2 dashboard
	- Under Recouses select security groups
	- Find your instance and open
	- Open and edit inbound rules
	- On SSH log change `custom` to `MyIP`
	- Save rules 
<br> </br>

- ---------------------------------------
### Setting up a new EC2 instance on AWS


- 1) Navigate to ec2 dashboard and create new instance
- 2) Choose Amazon Machine ubuntu server 16.04 (64-bit(x86) )
- 3) Choose an instance type, we chose Family:t2, Type:t2micro etc
- 4) Install instance details, change `subnet` to DevOpsStudent default 1a and `auto-assign public ip` to enable
- 5) Add storage default settings sufice
- 6) Add Tags, `key` = Name and `Value` = eng89_YourName_ExtraInfo
- 7) Config Security Group (set up to allow specific access), Create new one with naming convention eng89_name_SG_ExtraInfo . `type`=ssh,  `port range` = 22, `source` = MyIP (if you leave 
	- To have group access add another rule (inbound rules means you're allowing access to ppl coming in) and set: `type` = HTTP, `port` = 80, `source`= Anywhere
	- Because we have enabled public ip that means we need the http for anyone to access
- 8) Review instance launch. Just check everything is set up correctly 
- 9) LAUNCH 
- 10) `Choose an existing key pair`, `Select a key pair` = eng89_devops (This key-file has been sent to us by our supervisor)
- You will be given an instance ID. You can find your instance if you go to ec2 dashboard and type the name of your instance in `Filter instances`
<br> </br>
- ------------------------------------------------


### SSH into new instance
- (make sure you have the eng89_devops key)
<br> </br>
- Select your instance and click on `connect`. You will be provided with a few options on how to connect
- Go to `ssh client`
- Copy `chmod...` command to ensure your key is not publicaly viewable 
- Open Gitbash terminal and open directory where eng89_devops.pem key is, in my case it is in my ssh folder.
- Past `chmoc...` command in ssh directory. **Only required to do once for new instance**
- Now we need to connect to the machine using the key. Copy `example` line (`ssh -i etc.`) and paste in terminal. Allow continue connecting 
- (Adds key permanently into your hosts file in your machine)
<br> </br>
- -------------------------------------
### SSH into existing machine
- Find the instance, select it and press `connect`
- Copy and past the command in `ssh` directory
<br> </br>
- --------------------------
### Start/Stop instance
- Select an instance and select `instance state` and either select `Start instance` OR `Stop instance`

<br> </br>
- -------------------------
### Copy files from host to instance
- Copy app folder from: `/c/Users/Niki/eng89_virtualisation/Vagrant` to Instace
<br> </br>
- In `/vagrant` directory, run:  
` scp -i ~/.ssh/eng89_devops.pem -r app/ubuntu@54.73.28.131:~/app/`
<br> </br>
- Where:
	- `scp` securly copies files 
	- `~/.ssh...` is path to where to fetch the file
	- `-r` copy all items
	- `app` from this current location
	- `ubuntu...` copy to specific instance
	- `:~/app/` where you want to copy it
- Once copy complete, you should have app folder in the instance
<br> </br>
- -------------------------------------
### 2 tier architecture Sparta Global app setup
- Set up 2 instances,named app and db and follow steps from repo: eng89_virtualisation
<br> </br>
**In app instance**
- Update and upgrade 
- `sudo apt-get update -y`
- `sudo apt-get upgrade -y`
<br> </br>
- Install git
- `sudo apt-get install git -y`
<br> </br>
- Install nodejs
- `sudo apt-get install 
python-software-properties -y`
- `curl -sL https://deb.nodesource.com/setup_12.x | sudo -E bash -`
- `sudo apt-get install nodejs -y`
<br> </br>
- Install pm2
- `sudo npm install pm2 -g`
<br> </br>

- ------------------------------------
- **To allow public access to port 3000**
- select `eng89_niki_app` instance in aws and click `security` -> `Security groups` -> `Edit inbound rules` -> `add rule` 
- Change `port range` = 3000, `source` = Anywhere IPv4
- save rule and refresh page


![app](https://user-images.githubusercontent.com/86292184/127358241-7ce6a9c3-d2a3-4b26-858e-08aa7a3b906b.PNG)



- ----------------------------------
**Run app**
<br></br>
- Open directoy `app` and run `npm start`
- -----------------------------------
### Reverse Proxy
- In `eng89_niki_app` instance, go to `cd /etc/nginx/sites-available`
- Change default file to:
```
server {
    listen 80;

    server_name _;
    location / {
        proxy_pass http://54.73.28.131:3000;
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection 'upgrade';
        proxy_set_header Host $host;
        proxy_cache_bypass $http_upgrade;
    }
}


``` 
- Check nginx config `sudo nginx -t`
- Restart nginx `sudo systemctl restart nginx`
- Go to app directory and run `npm start`
<br> </br>
- -------------------------------------
### Install mongodb on eng89_niki_db instance

```
# mongodb keys
sudo apt-key adv --keyserver hkp://keyserver.ubuntu.com:80 --recv D68FA50FEA312927 
echo "deb https://repo.mongodb.org/apt/ubuntu xenial/mongodb-org/3.2 multiverse" | sudo tee /etc/apt/sources.list.d/mongodb-org-3.2.list

sudo apt-get update -y
sudo apt-get upgrade -y

# Install mongod and multiple packages
sudo apt-get install -y --allow-downgrades mongodb-org=3.2.20 mongodb-org-server=3.2.20 mongodb-org-shell=3.2.20 mongodb-org-mongos=3.2.20 mongodb-org-tools=3.2.20


```
<br> </br>
- Go to `cd /etc` and change bindIp to 0.0.0.0 in mongod.conf
- `sudo nano mongod.conf
`
- Go back to home directory and run 
```
# To restart and enable changes
sudo systemctl restart mongod
sudo systemctl enable mongod

```
- Good practice to check if mongod is running with: `systemctl status mongod
`
<br> </br>
- ---------------------------------
### Create persistant variable in eng89_niki_app instance

- Make sure to use db public ip:34.243.86.240 (in this case) to connect db and app
- To create a persistance variable, run: `sudo echo export DB_HOST="mongodb://34.243.86.240:27017/posts" >> ~/.bashrc
`
- Need to run the source file to reload the information `source ~/.bashrc`
- To check if varaible exists, run `env` or `printenv DB_HOST` 
<br> </br>
- ------------------------------------------------------------------------
- **To give app access to db port 27017**
- Open `Security groups` for eng89_niki_db instance and `edit unbound rules`
- Create new rule and change `port range`=27017 and `source`=54.73.28.131/32, where the ip is the app ip
- save new rules

![db](https://user-images.githubusercontent.com/86292184/127358305-d324c88f-27d4-4abd-bf13-b865b25f17be.PNG)

- -------------------------------------
### Run app/posts
- In `app` directory open `seeds` and run `node seed.js`
- Go back to `app` directory, run `npm start` and all should work