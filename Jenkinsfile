pipeline {
    agent {
        label 'slave2'
    }
    tools {
        maven 'maven-3.8.9'
    }
    environment {
        APPLICATION_NAME = 'eureka'
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
                sh """
                    mvn clean verify sonar:sonar \
                        -Dsonar.projectKey=i27-eureka \
                        -Dsonar.host.url=http://34.30.55.27:9000 \
                        -Dsonar.login=sqp_783c785146fc4d053116b848c3b433047308f4f2
                """
            }
        }
    }
}