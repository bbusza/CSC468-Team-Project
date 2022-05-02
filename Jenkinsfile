pipeline {
    agent none
    environment {
        registry = "Every-Villain-Is-Lemons/CSC468-Team-Project"
        docker_user = "thescavenger126"
        GOCACHE = "/tmp"
    }
    stages {
        stage('Publish') {
            agent {
                kubernetes {
                    inheritFrom 'gokoins-webui'
                }
            }
            steps{
                container('docker') {
                    sh 'echo $DOCKER_TOKEN | docker login --username $DOCKER_USER --password-stdin'
                    sh 'docker build -t $DOCKER_REGISTRY:$BUILD_NUMBER -f webui/Dockerfile .'
                    sh 'docker push $DOCKER_REGISTRY:$BUILD_NUMBER'
                }
            }
        }
        stage ('Deploy') {
            agent {
                node {
                    label 'deploy'
                }
            }
            steps {
                sshagent(credentials: ['cloudlab']) {
                    sh 'scp -r -v -o StrictHostKeyChecking=no *.yml tylerp@clnodevm020-1.clemson.cloudlab.us:~/'
                    sh 'ssh -o StrictHostKeyChecking=no tylerp@clnodevm020-1.clemson.cloudlab.us kubectl create deployment webui --image=127.0.0.1:30000/webui:v0.1 -n jenkins'                    
                    sh 'ssh -o StrictHostKeyChecking=no tylerp@clnodevm020-1.clemson.cloudlab.us kubectl expose deploy/webui --type=NodePort --port=80 -n jenkins'
                    sh 'ssh -o StrictHostKeyChecking=no tylerp@clnodevm020-1.clemson.cloudlab.us kubectl apply -f dashboard-insecure.yml'
                    sh 'ssh -o StrictHostKeyChecking=no tylerp@clnodevm020-1.clemson.cloudlab.us kubectl apply -f socat.yml'
                    
                    // MANUAL: kubectl patch svc webui --type='json' --patch='[{"op": "replace", "path": "/spec/ports/0/nodePort", "value":30080}]' -n jenkins
                }
            }
        }
    }
}
