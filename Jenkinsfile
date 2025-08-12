pipeline {
    agent {
        label 'slave2'
    }
    tools {
        maven 'maven-3.8.9'
    }
    environment {
        POM_VERSION = readMavenPom().getVersion()
        POM_PACKAGING = readMavenPom().getPackaging()
        APPLICATION_NAME = 'eureka'
        SONAR_URL = 'http://34.30.55.27:9000'
        SONAR_TOKEN = credentials('sonar_creds')
        DOCKER_HUB = "docker.io/sundar9395"
        DOCKER_CREDS = credentials('docker_creds')
    }
    stages {
        stage('build') {
            steps {
                echo "my ${env.APPLICATION_NAME} application"
                sh "mvn clean package -DskipTests=true"
            }
        }
        stage('sonar') {
            steps {
                echo "sonar scans"
                withSonarQubeEnv('sonarQube') {
                    sh """
                         mvn clean verify sonar:sonar \
                              -Dsonar.projectKey=i27-eureka \
                              -Dsonar.host.url=${env.SONAR_URL} \
                              -Dsonar.login=${env.SONAR_TOKEN}
                    """
                }
                timeout (time: 2, unit: "MINUTES") {
                    script {
                        waitForQualityGate abortPipeline: true
                    }
                }
            }
        }
        stage ('formatbuild') {
            steps {
                echo "Testing JAR Source: i27-${env.APPLICATION_NAME}-${env.POM_VERSION}.${env.POM_PACKAGING}"
                echo "Testing JAR Destination: i27-${env.APPLICATION_NAME}-${BUILD_NUMBER}-${BRANCH_NAME}.${env.POM_PACKAGING}"
            }
        }
        stage ('dockerbuildandpush') {
            steps {
                echo "docker image pushing"
                sh "cp target/i27-${env.APPLICATION_NAME}-${env.POM_VERSION}.${env.POM_PACKAGING} ./.cicd"
                sh "docker build --no-cache --build-arg JAR_SOURCE=i27-${env.APPLICATION_NAME}-${env.POM_VERSION}.${env.POM_PACKAGING} -t ${env.DOCKER_HUB}/${env.APPLICATION_NAME}:$GIT_COMMIT ./.cicd"
                echo "docker login"
                sh "docker login -u ${DOCKER_CREDS_USR} -p ${DOCKER_CREDS_PSW}"
                echo "docker push"
                sh "docker push ${env.DOCKER_HUB}/${env.APPLICATION_NAME}:$GIT_COMMIT"
            }
        }
        stage ('Deploy To dev') {
            steps {
                script {
                    dockerDeploy('dev', '5761').call()
                }
            }
        }
        stage ('Deploy To test') {
            steps {
                script {
                    dockerDeploy('dev', '6761').call()
                }
            }
        }
        stage ('Deploy To stage') {
            steps {
                script {
                    dockerDeploy('dev', '7761').call()
                }
            }
        }
        stage ('Deploy To prod') {
            steps {
                script {
                    dockerDeploy('dev', '8761').call()
                }
            }
        }
    }
}

def dockerDeploy(envDeploy, port){
    return {
        echo "Deploying to $envDeploy Environment"
            withCredentials([usernamePassword(credentialsId: 'remo_docker_creds', passwordVariable: 'PASSWORD', usernameVariable: 'USERNAME')]) {
                    // some block
                script {
                    try {
                        // stop the container
                        sh "sshpass -p $PASSWORD -v ssh -o StrictHostKeyChecking=no $USERNAME@$docker_vm_ip \"docker stop ${APPLICATION_NAME}-$envDeploy\""

                        // remove the container
                        sh "sshpass -p $PASSWORD -v ssh -o StrictHostKeyChecking=no $USERNAME@$docker_vm_ip \"docker rm ${APPLICATION_NAME}-$envDeploy\"" 
                    }
                    catch(err) {
                        echo "Error caught: $err"
                    }
                    // Creating a container
                    sh "sshpass -p $PASSWORD -v ssh -o StrictHostKeyChecking=no $USERNAME@$docker_vm_ip \"docker run --name ${APPLICATION_NAME}-$envDeploy -p $port:8761 -d ${env.DOCKER_HUB}/${env.APPLICATION_NAME}:$GIT_COMMIT\""
                }
            } 
    }
}