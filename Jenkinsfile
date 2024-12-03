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
        }
        
        stage("mvn build") {
            steps {
                script {
                    // Run Maven build
                    sh 'mvn -Dmaven.test.failure.ignore=true install'
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
        }
    }

    post {
        always {
            // Send Slack notification at the end of the pipeline
            script {
                def status = currentBuild.result ?: 'SUCCESS'  // Default to SUCCESS if not set
                def message = ""
                def color = ""

                if (status == 'SUCCESS') {
                    message = "Pipeline completed successfully. All stages passed!"
                    color = 'good'  // Green for success
                } else {
                    message = "Pipeline failed. Check the logs for details."
                    color = 'danger'  // Red for failure
                }

                // Send the Slack notification
                slackSend(channel: SLACK_CHANNEL, message: message, color: color)
            }
        }
    }
}
