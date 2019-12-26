# Step-1 [One-time step to prepare docker images]
* Make changes in docker-compose.yml to build Mattermost TEAM edition
* mkdir -p ./volumes/app/mattermost/{data,logs,config,plugins}
* chown -R 2000:2000 ./volumes/app/mattermost/
* docker-compose build
	* Build docker images for db, web, app
* docker-compose up -d
	* Runs docker containers
* Push images to Amazon ECR

# Step-2 [Deploy to Elastic Beanstalk]
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

# Step-3 [Setup AWS Aurora-Serverless]
* Set Elastic Beanstalk environment variables
	* MM_SQLSETTINGS_DRIVERNAME: mysql
	* MM_SQLSETTINGS_DATASOURCE: {USERNAME}:{PASSWORD}@tcp(HOST:PORT)/{DB_NAME}?writeTimeout=30s&readTimeout=30s

* Setting these environment variables makes Mattermost access the specified DB

# Step-4 [Allow network traffic for Web Socket connection]
* In Elastic Beanstalk, modify load balancer. In previous step we have chosen classic load balancer
	* Modify from 80:HTTP to 80:TCP

# Step-5 [Set config]
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

# Step-6 [Email related settings]
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

# Step-7 [File storage settings]
* Set environment variables related to File storage
	* MM_FILESETTINGS_DRIVERNAME: amazons3
	* MM_FILESETTINGS_AMAZONS3ACCESSKEYID
	* MM_FILESETTINGS_AMAZONS3SECRETACCESSKEY
	* MM_FILESETTINGS_AMAZONS3BUCKET
	* MM_FILESETTINGS_AMAZONS3REGION

# Step-8 [Remove db container]
* As we no longer need db container, we remove it from the configuration file
