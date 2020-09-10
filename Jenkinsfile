pipeline {
    agent any

    stages {
        stage('clean') {
            steps {
                script{
                    try{
                        powershell "docker stop \$(docker ps --filter name=adminserver -qa)"
                        powershell "docker container rm \$(docker ps --filter name=adminserver -qa)"
                    }
                    catch(exc){
                        echo 'no container'
                    }
                    try {
                        powershell "docker image rm \$(docker image ls --filter reference=adminserver -qa)"
                    }
                    catch(exc){
                        echo 'no image'
                    }
                }
            }
        }
        stage('build') {
            steps {
                bat "mvn clean package"
            }
        }
        stage('sonar') {
            steps {
                bat "mvn sonar:sonar \
                      -Dsonar.projectKey=adminserver \
                      -Dsonar.host.url=http://localhost:1001 \
                      -Dsonar.login=8dbd1aa5aaecbf60711560ff1c458d4e0e3597fe"
            }
        }
        stage('docker') {
            steps {
                script{
                    image = docker.build("adminserver:${env.BUILD_ID}")
                    image.run("-p 8010:8010 --name adminserver.${env.BUILD_ID} --restart=always \
                            --add-host eurekaserver1:172.17.0.1 --add-host eurekaserver2:172.17.0.1")
                }
            }
        }
    }
}
