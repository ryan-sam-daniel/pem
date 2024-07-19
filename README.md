pipeline {
    agent any

    stages {
        stage('git checkout') {
            steps {
                git branch: 'main', url: 'https://github.com/ryan-sam-daniel/mvn-pipe-azure.git'
            }
        }
        stage('maven build') {
            steps {
                sh 'mvn clean package'
            }
        }
        stage('docker image') {
            steps {
                script{
                    withDockerRegistry(credentialsId: '3ad3fc66-79ae-46ce-ac88-c4b585cbed44', toolName: 'docker') {
                        try{
                            sh 'docker stop webapp'
                            sh 'docker rm webapp'
                            sh 'docker rmi ryansamdaniel/ci-pipeline'
                        }
                        catch(Exception e){
                            
                        }
                        
                        sh 'docker build -t ryansamdaniel/ci-pipeline -f Dockerfile .'
                        sh 'docker push ryansamdaniel/ci-pipeline'
                    }
                }
            }
        }
        stage('trigger cd') {
            steps {
                build job:'cd-pipeline',wait:true
            }
        }
    }
}
##ci 

#cd
pipeline {
    agent any

    stages {
        stage('docker ') {
            steps {
                script{
                    withDockerRegistry(credentialsId: '3ad3fc66-79ae-46ce-ac88-c4b585cbed44', toolName: 'docker') {
                        sh 'docker run --name webapp -d -p 8081:8080 ryansamdaniel/ci-pipeline:latest'
                    }
                }
            }
        }
    }
}
