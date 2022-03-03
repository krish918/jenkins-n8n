# jenkins-n8n
A simple Jenkinsfile to help build N8N and run workflow present in the repo.

## Jenkinsfile
It contains a three-stage pipeline.
  - **First Stage** :  Clones the N8N repo consisting of all Intel specific modifications, installs all essential tools needed and builds N8N.
  - **Second Stage** : This stage installs `docker` and `docker-compose` on the current node, if it is not already available. Then, it looks for `docker-compose.yml` file in current repo and spins-up the required services.
    
  - **Third Stage** : It helps import the credentials files needed by the workflow, and run N8N CLI to execute the workflow.

## docker-compose.yml
It contains docker-compose instructions to spin-up 2 containers needed by the N8N *workflow files* present in this repo.
- `dlstreamer-pipeline-server` : Needed by Intel VAS node running in the workflow. 
- `eclipse-mosquitto` : Helps in running a MQTT broker, which is needed by Intel VAS during run-time.

## mosquitto.conf
A configuration file needed by `eclipse-mosquitto` container.

## Workflow files
`workflow.json`, `credentials.json` and `config` are files needed by N8N to run the workflow.
