pipeline {
    agent any

    tools {
        maven 'M2_HOME'
        jfrog 'Jfrog remote cli'
    }

    stages {
        stage('Checkout') {
            steps {
                git branch: 'main', url: 'https://github.com/alixstearns/geoapp.git'
            }
        }
        stage('Sonarqube scan') {
            steps {
                withSonarQubeEnv('sonarQube') {
        sh 'mvn verify org.sonarsource.scanner.maven:sonar-maven-plugin:sonar -Dsonar.projectKey=alixstearns_geoapp'
                }
            }
        }
        stage('Build and Test') {
            steps {                 
                script { 
         // Run Maven commands 
        sh 'mvn clean install compile test package'
                } 
            }
        } 

        stage('Testing1') {
            steps {
                // Show the installed version of JFrog CLI.
                jf '-v'
                // Show the configured JFrog Platform instances.
                jf 'c show'
                // Ping Artifactory.
                jf 'rt ping'
                // Create a file and upload it to a repository named 'my-test-repo' in Artifactory
                sh 'touch geo-app'
                jf 'rt u geo-app geoapp/'
                // Publish the build-info to Artifactory.
                jf 'rt bp'
                // Download the test-file
                jf 'rt dl geoapp/geo-app'
            }
        }
    }
}
