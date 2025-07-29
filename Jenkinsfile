pipeline {
    agent {
        label 'slave2'
    }
    tools {
        maven 'maven-3.8.9'
    }
    stages {
        stage ('buildstage') {
            steps {
                echo "building my eureka application"
                sh "mvn clean package"
            }
        }
    }
}