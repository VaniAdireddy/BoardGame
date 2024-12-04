pipeline {
    agent any
    tools {
        jdk 'jdk17'
        maven 'maven3'
    }
    environment {
        SCANNER_HOME = tool 'sonar-scanner'
        DOCKER_IMAGE = "vasanthaadireddy/boardshack:${env.BUILD_NUMBER}" // Image name with versioning
    }
    stages {
        stage('Git Checkout') {
            steps {
                git branch: 'prod', credentialsId: 'git-cred', url: 'https://github.com/VaniAdireddy/BoardGame.git'
            }
        }
        stage('Versioning') {
        steps {
        script {
            sh 'mvn versions:set -DnewVersion=1.0.${BUILD_NUMBER}'
            }
        }
        }
        stage('Compile') {
            steps {
                sh 'mvn compile'
                echo 'GITHUB Webhook Configured'
            }
        }
        stage('Test') {
            steps {
                sh 'mvn test'
            }
        }
        stage('File System Scan') {
            steps {
                sh 'trivy fs --format table --output trivy-fs-report.html .'
            }
        }
        stage('SonarQube Analsyis') {
            steps {
                withSonarQubeEnv('sonar') {
                    sh ''' $SCANNER_HOME/bin/sonar-scanner -Dsonar.projectName=BoardGame -Dsonar.projectKey=BoardGame \
                            -Dsonar.java.binaries=. -Dsonar.exclusions=**/trivy-image-report.html'''
                }
            }
        }
        stage('Quality Gate') {
            steps {
                script {
                    waitForQualityGate abortPipeline: false, credentialsId: 'sonar'
                }
            }
        }
        stage('Build') {
            steps {
                sh 'mvn package'
            }
        }
        stage('Publish To Nexus') {
            steps {
                withMaven(globalMavenSettingsConfig: 'global-settings', jdk: 'jdk17', maven: 'maven3', mavenSettingsConfig: '', traceability: true) {
                    sh 'mvn deploy -X'
                }
            }
        }
    }
}