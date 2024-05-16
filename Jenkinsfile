pipeline {
    agent any

    tools {
        jdk 'jdk17'
        maven 'maven3'
    }
    
    environment {
        SCANNER_HOME= tool 'sonar-scanner'
        DOCKERHUB_CREDENTIALS = credentials('docker-cred')
    }

    stages {
        stage('Git Checkout') {
            steps {
                git branch: 'main', credentialsId: 'git-cred', url: 'https://github.com/sami1895/Boardergame.git'
            }
        }
    stage('Compile') {
            steps {
                sh 'mvn compile'
            }
        }
        
    stage('Test') {
            steps {
                sh 'mvn test'
            }
        }
    
    stage('File System Scan') {
            steps {
                sh "trivy fs --format table -o trivy-fs-report.html ."
            }
        }
    
    stage('SonarQube Analsyis') {
            steps {
                withSonarQubeEnv('sonar') {
                    sh ''' $SCANNER_HOME/bin/sonar-scanner -Dsonar.projectName=Boardergame -Dsonar.projectkey=Boardergame \
                     -Dsonar.java.binaries=. '''
                }
            }
        }

    stage('Quality Gate') {
            steps {
                script {
                  waitForQualityGate abortPipeline: false, credentialsId: 'sonar-token'
                  
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
                    sh 'mvn deploy'
                }
            }
        }    
     
      stage('Docker Login') {
            steps {
                sh 'echo $DOCKERHUB_CREDENTIALS_PSW | docker login -u $DOCKERHUB_CREDENTIALS_USR --password-stdin'
            }
        }
      
      

    }
}

