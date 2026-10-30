pipeline {
    agent any
    tools { maven 'M3' }
    options {
        skipDefaultCheckout(true)
        timestamps()
        disableConcurrentBuilds()
        buildDiscarder(logRotator(numToKeepStr: '10'))
        timeout(time: 30, unit: 'MINUTES')
    }
    triggers { pollSCM('H/5 * * * *') }
    environment {
        NEXUS_URL = 'http://lab7-nexus:8081'
        NEXUS_REPO = 'maven-snapshots'
    }
    stages {
        stage('Checkout') { steps { checkout scm } }
        stage('Build') {
            steps { dir('sample-java-app') { sh 'mvn -B clean compile' } }
        }
        stage('Test') {
            steps { dir('sample-java-app') { sh 'mvn -B test' } }
            post { always { junit 'sample-java-app/target/surefire-reports/*.xml' } }
        }
        stage('SonarQube Analysis') {
            steps {
                dir('sample-java-app') {
                    withSonarQubeEnv('SonarQube') {
                        sh 'mvn -B sonar:sonar -Dsonar.projectKey=devops-lab7 -Dsonar.projectName="DevOps Lab 7 CI Demo"'
                    }
                }
            }
        }
        stage('Package & Archive') {
            steps {
                dir('sample-java-app') {
                    sh 'mvn -B package -DskipTests'
                    archiveArtifacts artifacts: 'target/*.jar', fingerprint: true
                }
            }
        }
        stage('Publish to Nexus') {
            steps {
                dir('sample-java-app') {
                    withCredentials([usernamePassword(credentialsId: 'nexus-credentials', usernameVariable: 'NEXUS_USERNAME', passwordVariable: 'NEXUS_PASSWORD')]) {
                        configFileProvider([configFile(fileId: 'maven-settings-nexus', variable: 'MAVEN_SETTINGS')]) {
                            sh '''
                                mvn -B deploy:deploy-file -s "$MAVEN_SETTINGS" \
                                  -DpomFile=pom.xml \
                                  -Dfile=target/ci-demo-1.0.0-SNAPSHOT.jar \
                                  -Durl="$NEXUS_URL/repository/$NEXUS_REPO/" \
                                  -DrepositoryId=nexus
                            '''
                        }
                    }
                }
            }
        }
    }
    post {
        success { echo 'PIPELINE SUCCESS: tests, scan, archive and publish completed.' }
        failure { echo 'PIPELINE FAILED: inspect Console Output.' }
        always { cleanWs() }
    }
}
