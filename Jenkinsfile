pipeline {
    agent any
    tools {
        maven 'MAVEN3'
        jdk 'OracleJDK8'
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
            steps {
                withSonarQubeEnv('${SONAR_SERVER}', envOnly: true) {
                    sh '${SONAR_SCANNER}/bin/sonar-scanner \
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
    }

}