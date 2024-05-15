pipeline {
    agent any
    tools {
        maven 'MAVEN3'
        jdk 'OracleJDK11'
    }
    environment {
        NEXUSIP = '172.31.51.193'
        NEXUSPORT = '8081'
        NEXUS_USER = 'admin'
        NEXUS_PASS = 'admin123'
        RELEASE_REPO = 'vpro-release'
        CENTRAL_REPO = 'vpro-central'
        SNAP_REPO = 'vpro-snapshot'
        NEXUS_GRP_REPO = 'vpro-group'
        MAVEN_SETTINGS = 'settings.xml'
        SONAR_SCANNER = 'sonarscanner'
        SONAR_SERVER = 'sonarserver'
        NEXUS_CREDENTIALS_ID = 'nexuslogin'
    }
    stages {
        stage('Build') {
            steps {
                sh 'mvn -s $MAVEN_SETTINGS -DskipTest clean install'
            }
            post {
                success {
                    archiveArtifacts artifacts: '**/target/*.war'
                }
            }
        }
        stage('Test') {
            steps {
                sh 'mvn -s $MAVEN_SETTINGS test'
            }
        }
        stage('Checkstyle analysis') {
            steps {
                sh 'mvn -s $MAVEN_SETTINGS checkstyle:checkstyle'
            }
        }
        stage('SonarQube analysis') {
            environment {
                SONAR_SCANNER_HOME = tool "${SONAR_SCANNER}"
            }
            steps {
                withSonarQubeEnv("${SONAR_SERVER}") {
                    sh '${SONAR_SCANNER_HOME}/bin/sonar-scanner \
                    -Dsonar.projectKey=vprofile \
                    -Dsonar.projectName=vprofile \
                    -Dsonar.projectVersion=1.0 \
                    -Dsonar.sources=src/ \
                    -Dsonar.java.binaries=target/test-classes/com/visualpathit/account/controllerTest/ \
                    -Dsonar.junit.reportPaths=target/surefire-reports/ \
                    -Dsonar.coverage.jacoco.xmlReportPaths=target/jacoco.exec \
                    -Dsonar.java.checkstyle.reportPaths=target/checkstyle-result.xml'
                }
            }
        }
        stage("Quality Gate") {
            steps {
                timeout(time: 5, unit: 'MINUTES') {
                    // Parameter indicates whether to set pipeline to UNSTABLE if Quality Gate fails
                    // true = set pipeline to UNSTABLE, false = don't
                    waitForQualityGate abortPipeline: true
                }
            }
        }
        stage('Upload artifacts Nexus') {
            steps {
                // Upload artifacts to Nexus
                nexusArtifactUploader(
                    nexusVersion: 'nexus3',
                    protocol: 'http',
                    nexusUrl: "${NEXUSIP}:${NEXUSPORT}",
                    groupId: 'QA',
                    version: "${env.BUILD_ID}-${BUILD_TIMESTAMP}",
                    repository: "${RELEASE_REPO}",
                    credentialsId: "${NEXUS_CREDENTIALS_ID}",
                    artifacts: [
                        [artifactId: 'vproapp',
                        classifier: '',
                        file: 'target/vprofile-v2.war',
                        type: 'war']
                    ]
                )
            }
        }
    }

}