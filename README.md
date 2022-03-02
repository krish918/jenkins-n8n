# jenkins-n8n
A simple Jenkinsfile to help build N8N and run workflows available in the repo.

## Jenkinsfile
It contains a three-stage pipeline.
  - **First Stage** :  Clones the N8N repo consisting of all Intel specific modifications, installs all essential tools needed and builds N8N.
  - **Second Stage** : ***TO BE IMPLEMENTED***. 
    This stage needs to install `docker` and `docker-compose` on the Jenkins server. Then, it should clone a sample repo at 
    https://github.com/krish918/dl-streamer-setup.
    
    This repo basically contains docker-compose instructions to spin-up 2 containers needed by the `workflow.json` file in the current repo. The first container `dl-streamer-pipeline-server`, is needed by Intel VAS node running in the workflow. Another container helps in running a MQTT broker, which is needed by Intel VAS during run-time.
    
  - **Third Stage** : It helps import the credentials files needed by the workflow, and run N8N CLI to execute the workflow.
