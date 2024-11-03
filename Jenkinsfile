pipeline {
    agent any 
    tools {
        maven 'maven3'
        jdk 'openjdk-11'
    }
    environment {
        // Project-specific variables from Jenkins environment
        SONAR_PROJECT_VERSION = "1.0"
        SONAR_SOURCES = "src/"
        SONAR_BINARIES = "target/classes"
        SONAR_PROJECT_KEY = "${env.SONAR_PROJECT_KEY}"
        SONAR_PROJECT_NAME = "${env.SONAR_PROJECT_NAME}"
        SONAR_SERVER = "sonarserver"
        SONAR_SCANNER = "sonarscanner"
        NEXUS_USER = "${env.NEXUS_USER}"
        NEXUS_PASS = "${env.NEXUS_PASS}"
        NEXUSIP = "${env.NEXUSIP}"
        NEXUSPORT = "${env.NEXUSPORT}"
        NEXUS_GRP_REPO = "${env.NEXUS_GRP_REPO}"
        SNAP_REPO = "${env.SNAP_REPO}"
        RELEASE_REPO = "proj-host-release"
    }

    stages {
        stage('Build Artifact') {
            steps {
                sh 'mvn install -DskipTests'
            }
            post {
                success {
                    archiveArtifacts artifacts: 'target/*.war'
                }
            }
        }
        
        stage('Test the Artifact') {
            steps {
                sh 'mvn test'
            }
        }

        stage('Checkstyle') {
            steps {
                sh 'mvn checkstyle:checkstyle'
            }
        }

        stage('SonarQube Analysis') {
            environment {
                scannerHome = tool "${SONAR_SCANNER}"
            }
            steps {
                withSonarQubeEnv("${SONAR_SERVER}") {
                    sh '''${scannerHome}/bin/sonar-scanner \
                        -Dsonar.projectKey=${SONAR_PROJECT_KEY} \
                        -Dsonar.projectName=${SONAR_PROJECT_NAME} \
                        -Dsonar.projectVersion=${SONAR_PROJECT_VERSION} \
                        -Dsonar.sources=${SONAR_SOURCES} \
                        -Dsonar.java.binaries=${SONAR_BINARIES} \
                        -Dsonar.junit.reportPaths=target/surefire-reports \
                        -Dsonar.jacoco.reportPaths=target/jacoco.exec \
                        -Dsonar.java.checkstyle.reportPaths=target/checkstyle-result.xml
                    '''
                }
            }
        }

        stage('Nexus Artifact Uploader') {
            steps {
                nexusArtifactUploader(
                    nexusVersion: 'nexus3',
                    protocol: 'http',
                    nexusUrl: "${NEXUSIP}:${NEXUSPORT}",
                    groupId: 'JA',
                    version: "${env.BUILD_ID}",
                    repository: "${RELEASE_REPO}",
                    credentialsId: 'nexus-login', // Replace with your Jenkins credentials ID for Nexus
                    artifacts: [
                        [
                            artifactId: "jenkans",
                            classifier: '',
                            file: "target/vprofile-v2.war",
                            type: 'war'
                        ]
                    ]
                )
            }
        }
    }
}
