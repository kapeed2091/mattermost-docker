# Step-1 [One-time step to prepare docker images]
* Make changes in docker-compose.yml to build Mattermost TEAM edition
* mkdir -p ./volumes/app/mattermost/{data,logs,config,plugins}
* chown -R 2000:2000 ./volumes/app/mattermost/
* docker-compose build
	* Build docker images for db, web, app
* docker-compose up -d
	* Runs docker containers
* Push images to Amazon ECR

# Step-2
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
	
