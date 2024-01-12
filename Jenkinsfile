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
        stage("Build & SonarCloud analysis") {
            steps {
                script {
                    withSonarQubeEnv('sonarQube') {
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

        stage('Testing') {
            steps {
                // Show the installed version of JFrog CLI
                jf '-v'

                // Show the configured JFrog Platform instances
                jf 'c show'

                // Ping Artifactory
                jf 'rt ping'

                // Create a file and upload it to the 'geoapp' repository in Artifactory
                sh 'touch test-file'
                jf 'rt u test-file geoapp/'

                // Publish the build-info to Artifactory
                jf 'rt bp'

                // Download the test-file from the 'geoapp' repository
                jf 'rt dl geoapp/test-file'
            }
        }
    }
}
