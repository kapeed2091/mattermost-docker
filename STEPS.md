# Step-1 [One-time step to prepare docker images]
* Make changes in docker-compose.yml to build Mattermost TEAM edition
* mkdir -p ./volumes/app/mattermost/{data,logs,config,plugins}
* chown -R 2000:2000 ./volumes/app/mattermost/
* docker-compose build
	* Build docker images for db, web, app
* docker-compose up -d
	* Runs docker containers
* Push images to Amazon ECR
