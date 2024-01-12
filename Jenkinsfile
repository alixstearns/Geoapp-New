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

        stage("Maven Build Back-End") {
            steps {
                echo 'Build Back-End Project...'
                script {
                    sh 'mvn package -DskipTests=true'
                }
            }
        }

        stage("Publish to JFrog Artifactory") {
            steps {
                echo 'Publish to JFrog Artifactory...'
                script {
                    def pom = readMavenPom file: 'pom.xml'
                    def filesByGlob = findFiles(glob: "target/*.${pom.packaging}")
                    def artifactPath = filesByGlob[0].path

                    rtUpload (
                        serverId: ARTIFACTORY_SERVER_ID,
                        spec: """{
                            "files": [{
                                "pattern": "${artifactPath}",
                                "target": "${ARTIFACTORY_REPO}/${pom.groupId}/${pom.artifactId}/${pom.version}/${filesByGlob[0].name}"
                            }]
                        }""",
                        deployerCredentialsConfig: "${ARTIFACTORY_CREDENTIAL_ID}"
                    )
                }
            }
        }

        // Add more stages as needed...

    }
}

