pipeline {
    agent any

    tools {

        maven 'M2_HOME'

        // Specifying JFrog CLI tool

        jfrog 'Jfrog remote cli'

    }
    stages {

        stage('maven clean') {

            steps {

                sh 'mvn clean'
            }
        }
        stage('maven install') {
            steps {
                sh 'mvn install'
            }
        }

        stage('maven compile') {
            steps {
                sh 'mvn compile'
            }
        }

        stage('maven test') {
            steps {
                sh 'mvn test'
            }
        }

        stage('maven package') {
            steps {
                sh 'mvn package'
            }
        }
        stage('SonarQube analysis') {
            steps {
                withSonarQubeEnv('SonarQube') {
                    sh 'mvn verify sonar:sonar'
                }
            }
        }
        // New Testing stage to use JFrog CLI

        stage('Testing') {
            steps {
                // Show the installed version of JFrog CLI
                jf '-v'
                
                // Show the configured JFrog Platform instances
                jf 'c show'

                // Ping Artifactory
                jf 'rt ping'

                // Create a file and upload it to the 'geoapp' repository in Artifactory
                sh 'touch geo-app'
                jf 'rt u geo-app geoapp/'

                // Publish the build-info to Artifactory
                jf 'rt bp'
                
                // Download the test-file from the 'geoapp' repository
                jf 'rt dl geoapp/geo-app'
            }
        }
    }
}
