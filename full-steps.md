PROJECT:: AWS 3-TIER WEB ARCHITECHTURE

STEP 1:
      >>Download code from Github from following link >code > download zip
         https://github.com/aws-samples/aws-three-tier-web-architecture-workshop
	  
	  
Step 2:
      >> create s3 bucket 
	     > create bucket > name > block all public access> everythink else default > create bucket
   
step 3:
      >>IAM ec2 role creation
   
         >serch bar-IAM > Role- create role >aws service >use case - EC2 >add permission - (AmazonSSMManagedInstanceCore & AmazonS3ReadOnlyAccess) > next > Role name >descriptions > create role
	
step 4:
      >> configuring VPC

         >  search bar-VPC > create vpc > vpc only > name-3tierAppVpc >IPv4 CIDR-10.0.0.0/16 > create vpc
	 
step 5:
      >>configuring Subnets
	     >Subnets > create subnet >subnet name-Public-web-subnet-AZ-1 > AZ-us-east-1a > subnet CIDR-10.0.0.0/24
							  >subnet name-Public-web-subnet-AZ-2 > AZ-us-east-1b > subnet CIDR-10.0.1.0/24
							  >subnet name-Private-App-Subnet-AZ-1 > AZ-us-east-1a > subnet CIDR-10.0.2.0/24
							  >subnet name-Private-App-Subnet-AZ-2 > AZ-us-east-1b > subnet CIDR-10.0.3.0/24
							  >subnet name-Private-DB-Subnet-AZ-1 > AZ-us-east-1a > subnet CIDR-10.0.4.0/24
							  >subnet name-Private-DB-Subnet-AZ-2 > AZ-us-east-1b > subnet CIDR-10.0.5.0/24  >create subnet
			
step 6:
      >> create internet gateway
	     > to give internet access for public subnets >internet gateway > create gateway > name - 3-tier-gateway->create gateway
	     >attach to vpc > in available vpc- our created vpc > attach internet GW
	 
step 7:
      >> create NAT gateway
	     >name > subnet- public-web-subnet-AZ-1 > connectivity type-public >allocate Elastic IP
	     >name > subnet- public-web-subnet-AZ-2 > connectivity type-public >allocate Elastic IP >create nat gw
	 
step 8:
      >>Routing configuration
	     >create route table > name >vpc >create route table  >>edit routes > add route > 0.0.0.0/0 >internet gateway- 3-tier-gateway >save changes
	     >subnet association > edit subnet association > Public-web-subnet-AZ-1 & public-web-subnet-AZ-2 >save association
	
step 9:
      >>create route table for Private app layer subnet
         >create route table > PrivateRouteTable-AZ1 > select our vpc > create route table  >>edit routes >add route >0.0.0.0/0 >NAT gateway- NAT-gw-AZ1 >save changes
         >subnet association > edit subnet association > Private-App-Subnet-AZ-1 >save association
   
step 10:
      >> create security groups

         >create SG > name -internet-facing-lb-sg > descriptions -External Loadbalancer SG > vpc-3tierAppVpc > inbound rule -add rule >HTTP -anywhere IPv4 >create SG
   
         >create SG > name -web-tier-sg > descriptions -SG for public instance in the web tier > vpc-3tierAppVpc > inbound rule -add rule >HTTP -source-custom-internet-facing-lb-sg >
																										                  -add rule > HTTP-source-my IP >>>create SG
																														  
         >create SG > name -internal-lb-sg > descriptions -Loadbalancing Security Group > vpc-3tierAppVpc > inbound rule -add rule >HTTP -source-custom-web-tier-sg >create SG
   
         >create SG > name -Privateinstance-sg > descriptions -Privateinstance-sg > vpc-3tierAppVpc > inbound rule -add rule >custom TCP -port range-4000-source-custom-internal-lb-sg
																											     --add rule > custom TCP-port range-4000>source-my IP >>>create SG
																											  
         >create SG > name -DB-SG > descriptions -database security group > vpc-3tierAppVpc > inbound rule -add rule >MySQL/Aurora -source-custom-privateinstance SG >create SG
																											  
step 11:
      >> database deployment
         >subnet group deplyment
         >Aurora and RDS > subnet group >Create DB subnet group > name-three-tier-db-subnet-group >descriptions-Subnet group for the database layer >vpc-our own vpc > AZ- select both AZ >subnets- select both DB subnets >create
   
         > create database >standard create >aurora(mysql compatible > template-dev/test >DB cluster identifier- database1 >master user name >self-managed->password  >instance configuration-r5large >Multi AZ deployment-create replice.. >vpc- own vpc > DB-subnet-sroup--three-tier-db-subnet-group
         >VPC security group- DB-SG  > untick -enalble performance insights ->everything else default > create
	 
step 12:
      >>App instance deployment
          >launch instance >name-MyAppserver1 >AMI-amazonlinux >KP-proceed without KP > network settings-edit >vpc-own vpc > subnet-private-app-subnet-AZ-1 >security group-privateinstance-sg >advance details-IAM instance profile- 3-tier-IAM-role >Launch instance
	      > connect-session manager >connect
		  >sudo -su ec2-user
		  >ping 8.8.8.8
		  >to stop-ctrl+c
		  >sudo wget https://dev.mysql.com/get/mysql57-community-release-el7-11.noarch.rpm
		  >sudo rpm --import https://repo.mysql.com/RPM-GPG-KEY-mysql-2022
		  >sudo yum install https://dev.mysql.com/get/mysql57-community-release-el7-11.noarch.rpm
		  >sudo yum install mysql -y
		  >mysql --version
		  
	  >>copy the RDS endpoint of Writer Instance.Run below commands
	  
	     "" mysql -h database-1-instance-1.ck322ukaqaik.us-east-1.rds.amazonaws.com -u k21 -p ""

         >CREATE DATABASE webappdb;
		 >SHOW DATABASES;
		 >USE webappdb;
		 
		 >CREATE TABLE IF NOT EXISTS transactions(id INT NOT NULL AUTO_INCREMENT, amount DECIMAL(10,2), description VARCHAR(100), PRIMARY KEY(id));
         >SHOW TABLES;
		 >INSERT INTO transactions (amount,description) VALUES ('400','groceries');
		 >SELECT * FROM transactions;
		 
STEP 13:
      >>open the application-code/app-tier/DbConfig.js file (FROM Downloaded ZIPfile)
	        modify with DB writer endpoint
			
	     module.exports = Object.freeze({
         DB_HOST : 'database-1-instance-1.ck322ukaqaik.us-east-1.rds.amazonaws.com',
         DB_USER : 'k21',
         DB_PWD : '*******',
         DB_DATABASE : 'webappdb'});
 
         >upload app-tier folder to s3.
  
step 14:
      >>installing necessary components to run our backend application
	     >install NVM 
	     > curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.38.0/install.sh | bash source ~/.bashrc
         >source ~/.bashrc
	     >nvm install 16
	     > nvm use 16
	  
	     >npm install -g pm2    (PM2is daemon process manager)
	     >npm install -g pm2
	  
	     >cd ~/
	  
	     >aws s3 cp s3://ccutul-3-tier-bucket/app-tier/ app-tier --recursive
	     >cd ~/app-tier/
	     >ls l
	     >npm install
	     >pm2 start index.js
	     >pm2 list
	     >pm2 logs
	     >Pm2 startup
	     >sudo env PATH=$PATH:/home/ec2-user/.nvm/versions/node/v16.20.2/bin /home/ec2-user/.nvm/versions/node/v16.20.2/lib/node_modules/pm2/bin/pm2 startup systemd -u ec2-user --hp /home/ec2-user
	    OR
	     >sudo env PATH=$PATH:/home/ec2-user/.nvm/versions/node/v16.20.2/bin /home/ec2-user/.nvm/versions/node/v16.20.2/lib/node_modules/pm2/bin/pm2 startup systemd -u ec2-user --hp /home/ec2-user
	     >pm2 save
	     >curl http://localhost:4000/health
	     >curl http://localhost:4000/transaction
	  
	  
step 15:
      >> Configuring Internal Load Balancer And Auto Scaling-image/AMI creation	  
	     >select EC2 instance > action -image and templated-create image > name-APPtierimage >description-app tier >uncheck-reboot instance >create
		 
step 16:  
      >>create target group for internal Loadbalancer
	    >create target group > instances >name- AppTiertargetgroup >HTTP-4000 >vpc-own > health checks-HTTP- /health >health threshold-2 >next> create target Group
	
	
	
step 17:
     >>internal load balancer
        >application load balancer -create >name-app-tier-internal-lb > scheme-internal >vpc-own > AZ & Subnets- us-east-1a= Private-App-Subnet-AZ-1 & us-east-1b=Private-App-Subnet-AZ-2 > SG-internal-lb-sg > listeners-HTTP-80-AppTiertargetgroup > create Loadbalancer
	 
step 18:
     >>launch template
        >  name-App-Tier-LaunchTemplate > description-App-Tier-LaunchTemplate > my AMIs-APPtierimage >t2micro >sg-privateinstance-sg >IAM role-3-tier-IAM-role >create launch template
	  
step 19:
     >> create auto scaling group
        >name -AppTierASg >  launch template- app-tier-launch-template > vpc > subnet-private-app-subnet-AZ-1 & private-app-subnet-AZ-2 >next >attach to existing LB > existing LB target group > desired capacity-2 > min & max=2 >keep default > create
	 

step 20:
      >>update Nginx.conf and upload to s3 with folder web-tier

        >copy LB dns name and paste to nginx.conf (line 58)

step 21:
      >>web instance deployment
      
	    >launch instanch > name  >amazon linux > vpc-own >subnet-public-web-subnet-AZ-1>sg-web-tier-sg >IAM role > launch instance
	    >connect >ssm >connect
	    >sudo -su ec2-user
	    >ping 8.8.8.8
	    >install all the necessary components needed to run our front-end application
	    >curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.38.0/install.sh | bash
	    >source ~/.bashrc
	    >nvm install 16
	    >nvm use 16
	    >cd ~/
	    >aws s3 cp s3://ccutul-3-tier-bucket/web-tier/ web-tier --recursive
	    >cd ~/web-tier
	    >npm install
	    >npm run build
	    >sudo yum install nginx
	    >cd /etc/nginx
	    >ls -l
	    >sudo cp nginx.conf nginx.conf_bkp
	    >sudo aws s3 cp s3://ccutul-3-tier-bucket/nginx.conf .
	    >sudo service nginx restart
	    >chmod -R 755 /home/ec2-user
	    >sudo chkconfig nginx on
	    we have Successfully deployed the Web Tier Instance
	  
step 22:
      >> web tier AMI
        >instance-mywebapp1-action-image&templates-create image > name-WebServerImage > uncheck reboot > create AMI

step 23:
      >> target group
        >create target group > instance > name-WebTierTargetGroup > vpc-own > health checks-HTTP-/health > create target group
	  
step 24:
      >>intenet facing load balancer
        >create LB >name-web-tier-external-lb > internet facing > vpc-own >subnets - public-web-subnet-AZ-1 & public-web-subnet-AZ-2 > SG-internet-facing-lb-sg > HTTP-80-WebTierTargetGroup >create load balancer
	  
step 25:
      >>launch template-for auto scaling
        >create launch template >name-WebServerLaunchTemplate > My AMIs-webserverimage > t2micro > SG-web-tier-sg > IAM role-3-tier-IAM-role >create launch template
	 
step 26:
     >>create auto scaling group
	    >name-WebServerASG > select launch template > next > vpc-own > AZ&subnets-us-east-1a=public-web-subnet-AZ-1 & us-east-1b=public-web-subnet-AZ-2 >next > attach to existing LB > existing LB target group-WebTierTargetGroup >next > desired capacity & Min & Max= 2 , 2 ,2 > create ASG
	 
step 27 :

     copy LB DNS name-web-tier-external-lb-562327514.us-east-1.elb.amazonaws.com	 
	 
	 PASTE INTO NEW BROWSER
	 
	 GIVES YOU FINAL RESULT
	  
	  
step 28: 
      
	 >>cleanup

       >delete Auto scaling groups
       >delete LB > delete listeners >	 delete LB
       >delete target group > select both >action >delete
       >delete launch template > action >delete
       >delete AMI >deregister and delete
       >terminate instances
       >Aurora&RDS >modify- to check if deletion protection should be unchecked > writer instance >action >delete me >  reader instance-action-delete me >DB cluster-action-uncheck create final snapshot >delete me
	   >delete NAT GW > action >delete
	   >delete elastic IP >release
	   >intenet GW > detach >delete
	   >route table >subnet association >uncheck subnets>save changes >routes >edit routes>remove >save changes >action >delete
	   >subnet > action > delete
	   >vpc >action >delete
	   >s3 > empty >delete
	   
	 
	  
	  