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

    environment {
        ARTIFACTORY_SERVER_ID = 'i-023acc47cff1309cb'
        ARTIFACTORY_REPO = 'geoapp'
        ARTIFACTORY_CREDENTIAL_ID = credentials('artifactory-userID')
        SONARCLOUD_TOKEN = credentials('sonarcloud-token-id')
    }

    stages {
        stage("Build & SonarCloud analysis") {
            steps {
                script {
                    withSonarQubeEnv('sonarcloud') {
                        sh 'mvn verify sonar:sonar'
                    }
                }
            }
        }

        stage('Check Quality Gate') {
            steps {
                echo 'Checking quality gate...'
                script {
                    timeout(time: 20, unit: 'MINUTES') {
                        def qg = waitForQualityGate()
                        if (qg.status != 'OK') {
                            error "Pipeline stopped because of quality gate status: ${qg.status}"
                        }
                    }
                }
            }
        }

        // ... (rest of your stages remain un
