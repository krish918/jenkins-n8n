SETUP_NEEDED = false
pipeline {
    agent {
        node {
            label 'aws-ec2'
        }
    }
    environment {
        NODEJS_DIR = "/usr/local/lib/nodejs"
        NODE_TAR_FILE = "node-v14.18.0-linux-x64.tar.gz"
        NODE_VER_BUILD = "node-v14.18.0-linux-x64"

        N8N_SETUP_DIR = "${JENKINS_HOME}/workspace/n8n-setup"
        N8N_HOME = "${JENKINS_HOME}/workspace/n8n"

        PROXY_FILE = "/etc/apt/apt.conf.d/00-proxy"
        VERIFY_PEER_CONFIG_FILE = "/etc/apt/apt.conf.d/99-verify-peer"

        __REPO_N8N = "https://github.com/krish918/n8n.git"
    }
    stages {
        stage("Build N8N") {
            steps {
                dir("${JENKINS_HOME}/workspace") {
                    script {
                        // check if N8N directory in the workspace already exists, otherwise clone it
                        if ( !fileExists (N8N_HOME) ) {
                            sh 'git clone "$__REPO_N8N"'

                            // if cloning the repo for first time, then setup is must-needed.
                            SETUP_NEEDED = true
                        }
                    }
                }
                dir( N8N_SETUP_DIR ) {
                    script {
                        /*
                         Check for a file which is created after BUILD completes succesfully. 
                         If this file is available, no need to re-run BUILD. 
                        */
                        if ( !fileExists ('./setup.conf') ) {

                            SETUP_NEEDED = true

                            // setup proxy setting file in order to run apt-get on the server
                            if ( !fileExists( PROXY_FILE ) ) {
                                sh 'echo "Acquire::http::proxy \\"http://proxy-dmz.intel.com:911\\";\nAcquire::https::proxy \\"http://proxy-dmz.intel.com:912\\";" >> "$PROXY_FILE"'           
                            }
                            
                            sh 'apt-get update -y'
                            
                            //fetch the node version recommended for BUILDING N8N
                            if ( !fileExists ( NODE_TAR_FILE ) ) {
                                sh 'apt-get install -y curl'
                                sh 'curl -O "https://nodejs.org/dist/v14.18.0/${NODE_TAR_FILE}"'
                            }
                            
                            // install nodejs
                            sh "mkdir -p ${NODEJS_DIR}"
                            dir_empty = sh ( script: 'test -z "$(ls -A $NODEJS_DIR)"', returnStatus: true ) == 0
                            if ( dir_empty == true ) {
                                sh 'apt-get install -y tar gzip'
                                sh "tar -xvzf ${NODE_TAR_FILE} -C ${NODEJS_DIR}"
                                sh 'chown -R $(whoami) "$NODEJS_DIR"' 
                            }

                            // install build tools
                            sh 'apt-get install -y build-essential python'

                            // set symbolic links for node and npm, to be available in PATH
                            npm_exist = sh ( script: 'test -L /usr/bin/npm', returnStatus: true ) == 0
                            if ( npm_exist == false ) {
                                sh "ln -s ${NODEJS_DIR}/${NODE_VER_BUILD}/bin/npm /usr/bin/npm"
                            }

                            node_exist = sh ( script: 'test -L /usr/bin/node', returnStatus: true ) == 0
                            if ( npm_exist == false ) {
                                sh "ln -s ${NODEJS_DIR}/${NODE_VER_BUILD}/bin/node /usr/bin/node"
                            }

                            // install lerna needed to build N8N
                            lerna_exist = sh ( script: 'test -L /usr/bin/lerna', returnStatus: true ) == 0
                            if ( lerna_exist == false ) {
                                sh 'npm install -g lerna'
                                sh "ln -s ${NODEJS_DIR}/${NODE_VER_BUILD}/bin/lerna /usr/bin/lerna"
                            }
                        }
                    }
                }
                dir( N8N_HOME ) {
                    script {
                        UPDATED = false
                        
                        // check if N8N repo has any new commits
                        if ( SETUP_NEEDED == false ) {
                            sh 'git remote update'
                            LOCAL = sh( script: 'git rev-parse @', returnStdout: true ).trim()
                            REMOTE = sh( script: 'git rev-parse @{u}', returnStdout: true ).trim()

                            // if there are new commits, pull updates
                            if ( LOCAL != REMOTE ) {
                                sh 'git pull origin master'
                                UPDATED = true
                            }
                        }

                        // if setup is needed or N8N repo is updated, start building N8N
                        if ( SETUP_NEEDED == true || UPDATED == true ) {
                            sh 'lerna bootstrap --hoist'
                            sh 'npm run build'
                            sh "$N8N_HOME/packages/cli/bin/n8n --version"

                            // after BUILD is succesful, create a file to make sure not to RE-BUILD unless required.
                            if ( !fileExists ("$N8N_SETUP_DIR/setup.conf")) {
                                sh 'echo "INITIAL_SETUP_DONE" >> "${N8N_SETUP_DIR}/setup.conf"'
                            }
                        }
                    }
                }
            }
        }
        
        stage("Setup Microservices") {
            steps {
                sh 'echo "TO DO"'
                /***
                * TO DO :  
                            1. install docker and start docker daemon
                            2. install docker-compose
                            3. run docker-compose inside CWD to fire up all the required containers
                                for this particular workflow.
                ***/

            }
        }
        stage("Import & Execute Workflow") {
            steps {

                //import credentials into N8N
                sh "$N8N_HOME/packages/cli/bin/n8n import:credentials --input=credentials.json"

                // copy the encryption key to current user's home dir
                sh 'cp config "${HOME}/.n8n/"'

                //execute workflow
                sh 'echo "Executing Workflow..."'
                sh "$N8N_HOME/packages/cli/bin/n8n execute --file workflow.json"
            }
        }
    }
}
