pipeline {
    agent any

    tools {
        jdk 'jdk17'
        maven 'maven3'
    }
    
    environment {
        SCANNER_HOME= tool 'sonar-scanner'
        DOCKER_CREDENTIALS_ID = 'docker-cred'
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

      stage('Build & Tag Docker Image') {
            steps {
                withDockerRegistry(credentialsId: 'docker-cred', toolName: 'docker') {
               
                    sh "docker build -t imas10/bordergame:latest ."
                }
            }
        }  
      stages {
        stage('Build & Tag Docker Image') {
            steps {
                script {
                    withDockerRegistry(credentialsId: env.DOCKER_CREDENTIALS_ID, toolName: 'docker') {
                        sh 'docker build -t imas10/boardshack:latest .'
                    }
                }
            }
        }
        
        stage('Docker Image Scan') {
            steps {
                script {
                    sh 'trivy image --format table -o trivy-fs-report.html imas10/boardshack:latest'
                }
            }
        }
        
        stage('Push Docker Image') { // Corrected stage name to "Push Docker Image"
            steps {
                script {
                    withDockerRegistry(credentialsId: env.DOCKER_CREDENTIALS_ID, toolName: 'docker') {
                        sh 'docker push imas10/boardshack:latest' // Corrected push command
                    }
                }
            }
        }

    }
}
