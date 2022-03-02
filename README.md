# jenkins-n8n
A simple Jenkinsfile to help build N8N and run workflows available in the repo.

## Jenkinsfile
It contains a three-stage pipeline.
  - **First Stage** :  Clones the N8N repo consisting of all Intel specific modifications, installs all essential tools needed and builds N8N.
  - **Second Stage** : ***TO BE IMPLEMENTED***. 
    This stage needs to install `docker` and `docker-compose` on the Jenkins server.
    
  - **Third Stage** : It helps import the credentials files needed by the workflow, and run N8N CLI to execute the workflow.

## docker-compose.yml
It contains docker-compose instructions to spin-up 2 containers needed by the `workflow.json` file. The first container `dlstreamer-pipeline-server`, is needed by Intel VAS node running in the workflow. Another container called `eclipse-mosquitto` helps in running a MQTT broker, which is needed by Intel VAS during run-time.

## mosquitto.conf
A configuration file needed by `eclipse-mosquitto` container.

## Other files
`workflow.json`, `credentials.json` and `config` are files needed by N8N to run the workflow.
