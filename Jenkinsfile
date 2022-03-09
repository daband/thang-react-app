pipeline{
    agent none 

    environment{
        CI = 'true'
        DOCKER_IMAGE = "daband20001809/thang-react-app"
    }

    stages{
        stage('Test'){
            agent{
               docker{
                   image 'node:14-stretch-slim'
                   args '-u 0:0 -v /tmp:/root/.cache'
               }                  
            }
            steps{
                sh 'npm install'
                sh 'chmod  777 ./scripts/test.sh'
                sh './scripts/test.sh'
            }           
        }
  
        stage('Build'){
            agent { node {label 'master'}}
            environment {
                DOCKER_TAG="${GIT_BRANCH.tokenize('/').pop()}-${GIT_COMMIT.substring(0,7)}"
            }
            steps{
                sh "docker build -t ${DOCKER_IMAGE}:${DOCKER_TAG} . "
                sh "docker tag ${DOCKER_IMAGE}:${DOCKER_TAG} ${DOCKER_IMAGE}:latest"
                sh "docker image ls | grep ${DOCKER_IMAGE}"
                withCredentials([usernamePassword(credentialsId: 'docker-hub', usernameVariable: 'DOCKER_USERNAME', passwordVariable: 'DOCKER_PASSWORD')]){
                    sh 'echo $DOCKER_PASSWORD | docker login --username $DOCKER_USERNAME --password-stdin'
                    sh "docker push ${DOCKER_IMAGE}:${DOCKER_TAG}"
                    sh "docker push ${DOCKER_IMAGE}:latest"
                }
                //clean to save disk
               sh "docker image rm ${DOCKER_IMAGE}:${DOCKER_TAG}"
               sh "docker image rm ${DOCKER_IMAGE}:latest"
            }
        }  

        stage('Deploy'){
            agent { node {label 'agent'}}
            steps{
                sh "kubectl --kubeconfig kubeconfig.yaml get nodes"
                sh "kubectl --kubeconfig kubeconfig.yaml apply -f deploy-app.yaml"
                sh "kubectl --kubeconfig kubeconfig.yaml apply -f service-app.yaml"
            }
        }
        
    }


    post{
        success{
            echo "Successfulllllll"
        }
        failure{
            echo "Failedddddddddd"
        }
    }
}