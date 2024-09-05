pipeline {
   agent any
   environment {
      REPOSITORY = '192.168.16.22:5000'
      PCC_CONSOLE_URL = "https://asia-southeast1.cloud.twistlock.com/aws-singapore-961150750"
      CONTAINER_NAME = "ubuntu"
      REGISTRY_PASS = 'admin'
      REGISTRY_USER = 'admin'
   }
    stages {
         stage('Clone repository') {
            steps {
                checkout scm
            }
         }      

         stage('Build') {
            steps {
               withDockerRegistry(credentialsId: 'Docker-hub', url: 'https://index.docker.io/v1/') {
                   // some block
                  sh 'docker build -t thaind91/ubuntu:v1.0 .'
                  sh 'docker push thaind91/ubuntu:v1.0 .'
                 // {                
                 // sh ''' 
                 // docker login -u $REGISTRY_USER -p $REGISTRY_PASS $REPOSITORY
                 // echo "Building the Docker image..."
                 // docker build -t $REPOSITORY/$CONTAINER_NAME:$BUILD_NUMBER .
                 // docker image ls
                 // '''
                }
            }
         }

         stage('Container Scan') {
            steps {
               script{
                  try {
                    prismaCloudScanImage ca: '', cert: '', dockerAddress: 'unix:///var/run/docker.sock', ignoreImageBuildTime: true, image: "$REPOSITORY/$CONTAINER_NAME:$BUILD_NUMBER", key: '', logLevel: 'debug', podmanPath: '', project: '', resultsFile: 'prisma-cloud-scan-results.json'
                  } finally {
                    prismaCloudPublish resultsFilePattern: 'prisma-cloud-scan-results.json'
                  }
               }
            }
         }         

         stage('Push Image') {
            steps {
                  sh ''' 
                  echo "Image push into registry"
                  docker push $REPOSITORY/$CONTAINER_NAME:$BUILD_NUMBER
                  '''
            }
         }

         stage('Container Sandbox Scan') {
            steps {
               withCredentials([usernamePassword(credentialsId: 'ssh_creds', passwordVariable: 'SSH_PASS', usernameVariable: 'SSH_USER')]) {
                  sh '''
                   mkdir -p ~/.ssh/
                   ssh-keyscan -t rsa,dsa 10.160.154.170 >> ~/.ssh/known_hosts
                   sshpass -p $SSH_PASS ssh $SSH_USER@10.160.154.170 'bash -s' <<EOF         
                   sudo chmod +x /home/sysadmin/apps/sandbox-scan.sh
                   sudo PCC_CONSOLE_URL=$PCC_CONSOLE_URL token=$token CONTAINER_NAME=$CONTAINER_NAME TAG=$BUILD_NUMBER /home/sysadmin/apps/sandbox-scan.sh
                   exit
                   EOF
                  '''
               }
            }
         }  
      }
   }
