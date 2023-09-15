# Getting started with the Elastic Stack and Docker-Compose

This repo is in reference to the blog [Getting started with the Elastic Stack and Docker-Compose](https://www.elastic.co/blog/getting-started-with-the-elastic-stack-and-docker-compose)

######################################################################################################
# Modified security group for palotest EC2 to allow inbound ports: 22/5601/19000/1514/2514
# Modified security group for windows bastin to allow outbound port: 19000
# Replaced logstash.conf with logstash-test.conf

######################################################################################################
# OS Install
sudo apt update && sudo apt upgrade -y
sudo apt install apt-transport-https ca-certificates curl software-properties-common jq vim
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -
sudo add-apt-repository "deb [arch=amd64] https://download.docker.com/linux/ubuntu focal stable"
sudo apt install docker-ce
sudo usermod -aG docker ${USER}
sudo curl -L https://github.com/docker/compose/releases/download/v2.17.2/docker-compose-linux-x86_64 -o /usr/local/bin/docker-compose
sudo chmod +x /usr/local/bin/docker-compose
docker-compose --version
sudo sysctl -w vm.max_map_count=262144
sudo apt install git
git clone https://github.com/cruiser2016/elastic-stack-docker-part-one.git

cd elastic-stack-docker-part-one/
# Change password and encryption key in .env file
vi .env
docker-compose up -d

# To remove all containers and their volumes
docker-compose down -v

# Configure index template for Cribl in Elasticsearch
## Add integration for Palo in an agent policy. No need to assign to an agent as we are using logstash.
## Verify index template "logs-panw.panos" has index pattern "logs-panw.panos*" which should match our logstash.conf

# Install syslog output plugin in logstash docker container
docker exec -it logstash /bin/bash
bin/logstash-plugin install logstash-output-syslog


# Add Cribl master/worker container in docker-compose.yml
# Configure Cribl with following:
## Change password for admin/admin
## Created quick connect from syslog source on port 2514 to elasticsearch destination on https://es01:9200_bulk/
## Add Palo pack in the middle as processing pipeline
## Clone index pattern to logs-palo-cribl