
# Creating a website

### Step 1 - Install a weberver
I choose to use Nginx as my webserver, there are others to choose from but Nginx is free.  Once logged into the VM, I ran `sudo yum update`  to make sure all the repos were up to date.  Then I ran `sudo yum install nginx` .  When the command was run, it stated

```

No match for argument: nginx
Error: Unable to find a match: nginx

```

Going to the Nginx Wiki you have to tell yum where to find the nginx repos by creating /etc/yum.repos.d/nginx.repo with the following content
	
	[nginx-stable]
	name=nginx stable repo
	baseurl=http://nginx.org/packages/centos/$releasever/$basearch/
	gpgcheck=1
	enabled=1
	gpgkey=https://nginx.org/keys/nginx_signing.key
	module_hotfixes=true
	
	[nginx-mainline]
	name=nginx mainline repo
	baseurl=http://nginx.org/packages/mainline/centos/$releasever/$basearch/
	gpgcheck=1
	enabled=0
	gpgkey=https://nginx.org/keys/nginx_signing.key
	module_hotfixes=true

I was then able to install nginx using the previous install command, ensuring to match the fingerprint with the one provided on the nginx website

### Step 2 - Starting the webserver

Since Nginx doesn't start automatically run the command 

`sudo systemctl start nginx`

Initially I got an error about a Nginx not starting due to a timeout. I then ran

`sudo systemctl status nginx` 

to see if there was any better error messages explaining why it had timed out.
![work project nginx status](https://user-images.githubusercontent.com/87891944/184750092-d81f0cb0-9ec2-4933-a4f2-cdf704d2bd69.PNG)

There were none so I just tried running the start command again and it worked. 
![work project nginx status active](https://user-images.githubusercontent.com/87891944/184750417-472b46ab-9345-4274-9ed1-f1c64bb0df67.PNG)



### Step 3 - Configure firewall and initial connection test

Since CentOS enables firewalls by default which block all inbound traffic to ports 80 and 443 there are a few commands to be run to make sure you can connect.

`sudo firewall-cmd --zone=public --permanent --add-service=http`

`sudo firewall-cmd --zone=public --permanent --add-service=https`

`sudo firewall-cmd --reload`

After those three commands are complete with a success I then connect to my web server IP address using chrome and getting the nginx test page.

### Step 4 - Securing the webserver

To ensure that the webserver can only be accessed by known safe sources we set a few more firewall rules.

`sudo firewall-cmd --permanent --zone=public -add-source=156.154.0.0/16`

This adds the neustar IP range as a safe source

`sudo firewall-cmd --zone=public --add-rich-rule='rule family="ipv4" source address="156.154.0.0/16" invert="True" drop'`

This will then drop all traffic to the webserver from IPs outside of the neustar range

The firewall must then be reloaded to make sure those changes are commited

`sudo firewall-cmd --reload`


To verify that all of this was done correctly I tried to access the IP address from my phone and get a "the site can't be reached" page.

But when accessing from my work computer with the VPN connected I was get the Nginx default page.  Once I disconnected from the VPN I get a "the site can't be reached" page.
