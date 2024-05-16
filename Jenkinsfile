pipeline {
    agent any

    tools {
        jdk 'jdk17'
        maven 'maven3'
    }
    
    environment {
        SCANNER_HOME= tool 'sonar-scanner'
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


        
    }
}
