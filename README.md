# 2tiera app deployment
## Setting up new AWS ec2 instance.

- Make sure to be logged in and have location set to ireland

<br> </br>


- ---------------------------------------
### Setting up a new EC2 instance on AWS


- 1) Navigate to ec2 dashboard and create new instance
- 2) Choose Amazon Machine ubuntu server 16.04 (64-bit(x86) )
- 3) Choose an instance type, we chose Family:t2, Type:t2micro etc
- 4) Install instance details, change `subnet` to DevOpsStudent default 1a and `auto-assign public ip` to enable
- 5) Add storage default settings sufice
- 6) Add Tags, `key` = Name and `Value` = eng89_YourName_ExtraInfo (In my case `eng89_niki_db`)
- 7) Config Security Group (set up to allow specific access), Create new one with naming convention eng89_name_SG_ExtraInfo (In my case eng89_niki_SG_mongodb). `type`=ssh,  `port range` = 22, `source` = MyIP (if you leave 
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
- Past `chmoc...` command in ssh directory
- Now we need to connect to the machine using the key. Copy `example` line (`ssh -i etc.`) and paste in terminal. Allow continue connecting 
- (Adds key permanently into your hosts file in your machine)
<br> </br>
- -------------------------------------
### SSH into existing machine
- Find the instance, select it and press `connect`
- Copy and past the