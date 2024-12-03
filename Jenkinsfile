pipeline {
    agent any

    tools {
        maven "MVN_HOME"
    }

    environment {
        NEXUS_VERSION = "nexus3"
        NEXUS_PROTOCOL = "http"
        NEXUS_URL = "13.54.202.55:8081"
        NEXUS_REPOSITORY = "customer"
        NEXUS_CREDENTIAL_ID = "nexus"
        SLACK_CHANNEL = '#jenkins-integration'  // replace with your Slack channel name
        SLACK_CREDENTIALS_ID = 'Slack'  // Your Jenkins Slack API token credentials
    }

    stages {
        stage("clone code") {
            steps {
                script {
                    // Clone the source code
                    git 'https://github.com/betawins/sabear_simplecutomerapp.git'
                }
            }
            post {
                success {
                    slackSend(channel: SLACK_CHANNEL, Success: "Code cloned successfully", color: 'good')
                }
                failure {
                    slackSend(channel: SLACK_CHANNEL, Fail: "Code cloning failed", color: 'danger')
                }
            }
        }
        
        stage("mvn build") {
            steps {
                script {
                    // Run Maven build
                    sh 'mvn -Dmaven.test.failure.ignore=true install'
                }
            }
            post {
                success {
                    slackSend(channel: SLACK_CHANNEL, Success: "Maven build completed successfully", color: 'good')
                }
                failure {
                    slackSend(channel: SLACK_CHANNEL, Fail: "Maven build failed", color: 'danger')
                }
            }
        }
        
        stage("publish to nexus") {
            steps {
                script {
                    // Read POM and find the artifact
                    pom = readMavenPom file: "pom.xml"
                    filesByGlob = findFiles(glob: "target/*.${pom.packaging}")
                    echo "${filesByGlob[0].name} ${filesByGlob[0].path} ${filesByGlob[0].directory} ${filesByGlob[0].length} ${filesByGlob[0].lastModified}"
                    artifactPath = filesByGlob[0].path
                    artifactExists = fileExists artifactPath

                    if (artifactExists) {
                        echo "*** File: ${artifactPath}, group: ${pom.groupId}, packaging: ${pom.packaging}, version ${pom.version}"
                        nexusArtifactUploader(
                            nexusVersion: NEXUS_VERSION,
                            protocol: NEXUS_PROTOCOL,
                            nexusUrl: NEXUS_URL,
                            groupId: pom.groupId,
                            version: pom.version,
                            repository: NEXUS_REPOSITORY,
                            credentialsId: NEXUS_CREDENTIAL_ID,
                            artifacts: [
                                [artifactId: pom.artifactId, classifier: '', file: artifactPath, type: pom.packaging],
                                [artifactId: pom.artifactId, classifier: '', file: "pom.xml", type: "pom"]
                            ]
                        )
                    } else {
                        error "*** File: ${artifactPath}, could not be found"
                    }
                }
            }
            post {
                success {
                    slackSend(channel: SLACK_CHANNEL, Success: "Artifact published to Nexus successfully", color: 'good')
                }
                failure {
                    slackSend(channel: SLACK_CHANNEL, Fail: "Failed to publish artifact to Nexus", color: 'danger')
                }
            }
        }
    }

    post {
        always {
            slackSend(channel: SLACK_CHANNEL, Checking: "Pipeline finished", color: 'warning')
        }
        success {
            slackSend(channel: SLACK_CHANNEL, Success: "Pipeline completed successfully", color: 'good')
        }
        failure {
            slackSend(channel: SLACK_CHANNEL, Fail: "Pipeline failed", color: 'danger')
        }
    }
}
