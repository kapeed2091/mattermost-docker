# Step-0: Setting up base repo to begin with 
* git clone https://github.com/kapeed2091/mattermost-docker.git
* cd mattermost-docker
* git checkout b9484f444424057434fd8c0a1c1a937154639857

* We checkout to a previous version of code and make step-by-step changes as listed below

# [Step-1: One-time step to prepare docker images](https://github.com/kapeed2091/mattermost-docker/commit/b4c7fd67e64d8f634c5f12a944e27d335101383c)
* Make changes in docker-compose.yml to build Mattermost TEAM edition
* mkdir -p ./volumes/app/mattermost/{data,logs,config,plugins}
* chown -R 2000:2000 ./volumes/app/mattermost/
* docker-compose build
	* Build docker images for db, web, app
* docker-compose up -d
	* Runs docker containers
* Push images to Amazon ECR

# [Step-2: Deploy to Elastic Beanstalk](https://github.com/kapeed2091/mattermost-docker/commit/82329c1ec1020e81e336417cc486d6188e3fd145)
* chmod 777 contrib/aws/app/mattermost/config/config.json
* Go to AWS-ElasticBeanstalk
* Create a new environment
	* Environment name
	* Platform: Multi-container Docker
	* Prepare zip of contents in contrib/aws folder and upload
	* Configure more options: custom configuration
		* Modify capacity: 
			* Environment type: load balanced
			* Instances: Min(1), Max(1)
			* Instance type: micro
		* Modify load balancer: Classic load balancer
		* Modify security: Select EC2 key pair

# [Step-3: Setup AWS Aurora-Serverless](https://github.com/kapeed2091/mattermost-docker/commit/358902e1e686da5e4e707ef553af77680ee9886c)
* Set Elastic Beanstalk environment variables
	* MM_SQLSETTINGS_DRIVERNAME: mysql
	* MM_SQLSETTINGS_DATASOURCE: {USERNAME}:{PASSWORD}@tcp(HOST:PORT)/{DB_NAME}?writeTimeout=30s&readTimeout=30s

* Setting these environment variables makes Mattermost access the specified DB

# [Step-4: Allow network traffic for Web Socket connection](https://github.com/kapeed2091/mattermost-docker/commit/d5153bf298b1b7b44616a41fe59d48bf18370290)
* In Elastic Beanstalk, modify load balancer. In previous step we have chosen classic load balancer
	* Modify from 80:HTTP to 80:TCP

# [Step-5: Set config](https://github.com/kapeed2091/mattermost-docker/commit/5c3f227d316296af05a0a7d4cf5321fd729af164)
* Set other relevant environment variables
	* MM_SERVICESETTINGS_SITEURL
	* MM_SERVICESETTINGS_WEBSOCKETURL
	* MM_SERVICESETTINGS_ENABLEEMAILINVITATIONS: true
	* MM_TEAMSETTINGS_SITENAME
	* MM_TEAMSETTINGS_ENABLECUSTOMBRAND: true
	* MM_TEAMSETTINGS_CUSTOMBRANDTEXT
	* MM_TEAMSETTINGS_CUSTOMDESCRIPTIONTEXT
	* MM_SUPPORTSETTINGS_TERMSOFSERVICELINK
	* MM_SUPPORTSETTINGS_PRIVACYPOLICYLINK
	* MM_SUPPORTSETTINGS_ABOUTLINK
	* MM_SUPPORTSETTINGS_HELPLINK
	* MM_SUPPORTSETTINGS_REPORTAPROBLEMLINK
	* MM_SUPPORTSETTINGS_SUPPORTEMAIL

# [Step-6: Email related settings](https://github.com/kapeed2091/mattermost-docker/commit/64ac691dc121049e57882bb63b5ad5f291c0f98f)
* Set environment variables related to Email
	* MM_EMAILSETTINGS_FEEDBACKNAME
	* MM_EMAILSETTINGS_FEEDBACKEMAIL
	* MM_EMAILSETTINGS_FEEDBACKORGANIZATION
	* MM_EMAILSETTINGS_REPLYTOADDRESS
	* MM_EMAILSETTINGS_ENABLESMTPAUTH: true
	* MM_EMAILSETTINGS_SMTPUSERNAME
	* MM_EMAILSETTINGS_SMTPPASSWORD
	* MM_EMAILSETTINGS_SMTPSERVER
	* MM_EMAILSETTINGS_SMTPPORT: 465
	* MM_EMAILSETTINGS_CONNECTIONSECURITY: TLS

# [Step-7: File storage settings](https://github.com/kapeed2091/mattermost-docker/commit/2469deb7d538cf0f14fdd5b54b66665cae547b3a)
* Set environment variables related to File storage
	* MM_FILESETTINGS_DRIVERNAME: amazons3
	* MM_FILESETTINGS_AMAZONS3ACCESSKEYID
	* MM_FILESETTINGS_AMAZONS3SECRETACCESSKEY
	* MM_FILESETTINGS_AMAZONS3BUCKET
	* MM_FILESETTINGS_AMAZONS3REGION

# [Step-8: Remove db container](https://github.com/kapeed2091/mattermost-docker/commit/ec2843fc31db4d8d3c34042d7206d1ab900936b7)
* As we no longer need db container, we remove it from the configuration file

# [Step-9: TLS + Custom Domain](https://github.com/kapeed2091/mattermost-docker/commit/7fe54f667c944987d4d4cfc833bf2c0e35284b70)
* Add custom domain in Route53 (Ex: monthint.in)
* Generate certificates with LetsEncrypt for the domain name (Ex: *.monthint.in)
* Add files 'cert.pem' (fullchain.pem from LetsEncrypt), 'key-no-password.pem' (privkey.pem from LetsEncrypt) to contrib/aws/web/cert folder
* Add EBS URL to Route53 (Name: ibchat.monthint.in --> IPV4 Address Alias of EBS URL)
* Modify EBS load balancer to allow network traffic for port 443 (TCP:443)
* Modify environment variables MM_SERVICESETTINGS_SITEURL (Ex: https://ibchat.monthint.in), MM_SERVICESETTINGS_WEBSOCKETURL (Ex: wss://ibchat.monthint.in)

* LetsEncrypt certificates have to be renewed periodically
