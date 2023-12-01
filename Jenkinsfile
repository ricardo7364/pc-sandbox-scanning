pipeline {
   agent any
   environment {
      AWS_REGION = 'us-east-1'
      ECR_REPOSITORY = '603779782049.dkr.ecr.us-east-1.amazonaws.com/code2cloud-ecr'
      CONTAINER_NAME = 'code2cloud'
      PCC_SAN = 'us-east1.cloud.twistlock.com'
      // ECR_REPOSITORY = '' // ex. '89647xxxxxxx.dkr.ecr.us-east-1.amazonaws.com/code2cloud-ecr'
      // CONTAINER_NAME = 'code2cloud'
      // PCC_SAN = '' // 'us-east1.cloud.twistlock.com'
   }
    stages {
        stage('Build') {
            steps {
                withAWS(credentials: 'aws-cred', region: 'us-east-1') {
                  sh ''' 
                  docker ps
                  aws ecr get-login-password --region $AWS_DEFAULT_REGION | docker login --username AWS --password-stdin $ECR_REPOSITORY:$CONTAINER_NAME
                  echo "Building the Docker image..."
                  docker build -t $ECR_REPOSITORY:$CONTAINER_NAME ./app/
                  docker image  prune -f
                  docker image ls
                  '''
               }
            }
        }
      //   stage('Container Scan') {
      //    steps {
      //          withCredentials([
      //             string(credentialsId: 'PCC_CONSOLE_URL', variable: 'PCC_CONSOLE_URL'),
      //             string(credentialsId: 'PRISMA_ACCESS_KEY', variable: 'PRISMA_ACCESS_KEY'),
      //             string(credentialsId: 'PRISMA_SECRET_KEY', variable: 'PRISMA_SECRET_KEY')
      //             ]) {
      //             sh '''
      //             #This  command will generate an authorization token (Only valid for 1 hour)
      //             json_auth_data="$(printf '{ "username": "%s", "password": "%s" }' "${PRISMA_ACCESS_KEY}" "${PRISMA_SECRET_KEY}")"

      //             token=$(curl -sSLk -d "$json_auth_data" -H 'content-type: application/json' "$PCC_CONSOLE_URL/api/v1/authenticate" | python3 -c 'import sys, json; print(json.load(sys.stdin)["token"])')

      //             twistcli images scan --address $PCC_CONSOLE_URL --token=$token --details $ECR_REPOSITORY:$CONTAINER_NAME
      //             '''
      //          }
      //       }
      //   }

      // stage('IaC') {
      //    steps {
      //          withCredentials([
      //             string(credentialsId: 'PRISMA_ACCESS_KEY', variable: 'PRISMA_ACCESS_KEY'),
      //             string(credentialsId: 'PRISMA_SECRET_KEY', variable: 'PRISMA_SECRET_KEY')
      //             ]) {
      //             sh '''export PRISMA_API_URL=https://api2.prismacloud.io
      //               virtualenv -p python3 .venv
      //               . .venv/bin/activate && pip install bridgecrew
      //               . .venv/bin/activate && bridgecrew --directory . --skip-path .venv --bc-api-key $PRISMA_ACCESS_KEY::$PRISMA_SECRET_KEY --use-enforcement-rules --repo-id qaswqaa/code2cloud_cloud_breach
      //               '''
      //          }
      //       }
      //   }
        stage('Image Deploy') {
            steps {
                withAWS(credentials: 'aws-cred', region: 'us-east-1') {
                  sh ''' 
                  docker ps
                  $(aws ecr get-login --region $AWS_DEFAULT_REGION --no-include-email)
                  echo "Image push into registry"
                  docker push $ECR_REPOSITORY:$CONTAINER_NAME
                  '''
               }
            }
        }
        stage('Container Sandbox Scan') {
         steps {
            sshagent(credentials: ['ssh']) {
               withCredentials([
                  string(credentialsId: 'PCC_CONSOLE_URL', variable: 'PCC_CONSOLE_URL'),
                  string(credentialsId: 'PRISMA_ACCESS_KEY', variable: 'PRISMA_ACCESS_KEY'),
                  string(credentialsId: 'PRISMA_SECRET_KEY', variable: 'PRISMA_SECRET_KEY')
                  ]) {
                  sh '''
                  #This  command will generate an authorization token (Only valid for 1 hour)
                  json_auth_data="$(printf '{ "username": "%s", "password": "%s" }' "${PRISMA_ACCESS_KEY}" "${PRISMA_SECRET_KEY}")"

                  token=$(curl -sSLk -d "$json_auth_data" -H 'content-type: application/json' "$PCC_CONSOLE_URL/api/v1/authenticate" | python3 -c 'import sys, json; print(json.load(sys.stdin)["token"])')

                  [ -d ~/.ssh ] || mkdir ~/.ssh && chmod 0700 ~/.ssh
                  ssh-keyscan -t rsa,dsa 10.10.10.104 >> ~/.ssh/known_hosts
                  ssh ubuntu@10.10.10.104 'bash -s' <<EOF
                  
                  chmod +x /home/ubuntu/sandbox.sh
                  sudo PCC_CONSOLE_URL=$PCC_CONSOLE_URL token=$token ECR_REPOSITORY=$ECR_REPOSITORY CONTAINER_NAME=$CONTAINER_NAME /home/ubuntu/sandbox.sh

                  exit
                  EOF
                  '''
               }
            }
         }
      //   }
        stage('Server Deploy') {
         steps {
          sshagent(credentials: ['ssh']) {
            withCredentials([
                  string(credentialsId: 'PCC_CONSOLE_URL', variable: 'PCC_CONSOLE_URL'),
                  string(credentialsId: 'PRISMA_ACCESS_KEY', variable: 'PRISMA_ACCESS_KEY'),
                  string(credentialsId: 'PRISMA_SECRET_KEY', variable: 'PRISMA_SECRET_KEY')
                  ]) {
               sh '''
                  #This  command will generate an authorization token (Only valid for 1 hour)
                  json_auth_data="$(printf '{ "username": "%s", "password": "%s" }' "${PRISMA_ACCESS_KEY}" "${PRISMA_SECRET_KEY}")"

                  token=$(curl -sSLk -d "$json_auth_data" -H 'content-type: application/json' "$PCC_CONSOLE_URL/api/v1/authenticate" | python3 -c 'import sys, json; print(json.load(sys.stdin)["token"])')

                  [ -d ~/.ssh ] || mkdir ~/.ssh && chmod 0700 ~/.ssh
                  ssh-keyscan -t rsa,dsa 10.10.10.103 >> ~/.ssh/known_hosts
                  ssh ubuntu@10.10.10.103 'bash -s' <<EOF
                  # install prisma defender
                  
                  chmod +x /home/ubuntu/script.sh
                  sudo PCC_SAN=$PCC_SAN ECR_REPOSITORY=$ECR_REPOSITORY CONTAINER_NAME=$CONTAINER_NAME PCC_URL=$PCC_CONSOLE_URL token=$token /home/ubuntu/script.sh
                  exit
                  EOF
               '''
               }
          }
         }
        }
    }
}
