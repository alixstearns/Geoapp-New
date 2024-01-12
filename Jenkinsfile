pipeline {
    triggers {
        pollSCM('* * * * *')
    }
    agent any
    tools {
        maven 'M2_HOME'
        // Specifying JFrog CLI tool
        jfrog 'Jfrog remote cli'
    }

    stages {
        stage('Checkout') {
            steps {
                script {
                    // Checkout the source code
                    checkout scm
                }
            }
        }

        stage('Build and Test') {
            steps {
                script {
                    // Run Maven commands
                    sh 'mvn clean install compile test package sonar:sonar'
                }
            }
        }

        stage('Publish to Artifactory') {
            steps {
                script {
                    // Publish JAR file to JFrog Artifactory
                    sh 'jf rt u target/*.jar geoapp/'
                }
            }
        }
    }

    post {
        success {
            echo 'Pipeline successfully executed!'
        }
        failure {
            echo 'Pipeline failed. Check logs for details.'
        }
    }
}
