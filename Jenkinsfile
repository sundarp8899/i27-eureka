pipeline {
    agent {
        label 'slave2'
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