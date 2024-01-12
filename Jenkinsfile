
pipeline {
    triggers {
        pollSCM('* * * * *')
    }
    agent any
    tools {
        maven 'M2_HOME'
    }

    environment {
        ARTIFACTORY_SERVER_ID = 'i-023acc47cff1309cb'
        ARTIFACTORY_REPO = 'geoapp'
        ARTIFACTORY_CREDENTIAL_ID = 'artifactory-userID'
    }

    stages {
        stage("build & SonarQube analysis") {
            steps {
                script {
                    withSonarQubeEnv('sonar') {
                        sh 'mvn verify org.sonarsource.scanner.maven:sonar-maven-plugin:sonar -Dsonar.projectKey=kserge2001_geo'
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
                    sh "mvn package -DskipTests=true"
                }
            }
        }

        stage("Publish to JFrog Artifactory") {
            steps {
                echo 'Publish to JFrog Artifactory...'
                script {
                    pom = readMavenPom file: "pom.xml";
                    filesByGlob = findFiles(glob: "target/*.${pom.packaging}");
                    echo "${filesByGlob[0].name} ${filesByGlob[0].path} ${filesByGlob[0].directory} ${filesByGlob[0].length} ${filesByGlob[0].lastModified}"
                    artifactPath = filesByGlob[0].path;
                    artifactExists = fileExists artifactPath;
                    if (artifactExists) {
                        echo "*** File: ${artifactPath}, group: ${pom.groupId}, packaging: ${pom.packaging}, version ${pom.version}";
                        rtUpload (
                            serverId: ARTIFACTORY_SERVER_ID,
                            spec: """{
                                "files": [{
                                    "pattern": "${artifactPath}",
                                    "target": "${ARTIFACTORY_REPO}/${pom.groupId}/${pom.artifactId}/${pom.version}/${filesByGlob[0].name}"
                                }]
                            }""",
                            deployerCredentialsConfig: "${ARTIFACTORY_CREDENTIAL_ID}"
                        );
                    } else {
                        error "*** File: ${artifactPath}, could not be found";
                    }
                }
            }
        }
    }
}
