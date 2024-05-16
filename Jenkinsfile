pipeline {
    agent any

    tools {
        jdk 'jdk17'
        maven 'maven3'
    }
    
    environment {
        APP_NAME = "boardergame"
	SCANNER_HOME= tool 'sonar-scanner'
        DOCKERHUB_CREDENTIALS = credentials('docker-cred')
        RELEASE = "1.0.0"
	DOCKER_USER = "imas10"
        DOCKER_PASS = 'docker-cred'
	IMAGE_NAME = "${DOCKER_USER}" + "/" + "${APP_NAME}"
	IMAGE_TAG = "${RELEASE}-${BUILD_NUMBER}"
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


        
     
      stage('Docker Login') {
            steps {
                sh 'echo $DOCKERHUB_CREDENTIALS_PSW | docker login -u $DOCKERHUB_CREDENTIALS_USR --password-stdin'
            }
        }

       stage("Build & Push Docker Image") {
              steps {
                  script {
                     def dockerImage = docker.build("${IMAGE_NAME}")
                     dockerImage.push("${IMAGE_TAG}")
                     dockerImage.push('latest')
                }
            }
           }

	stage ('Cleanup Artifacts') {
           steps {
               script {
                    sh "docker rmi ${IMAGE_NAME}:${IMAGE_TAG}"
                    sh "docker rmi ${IMAGE_NAME}:latest"
               }
          }
       }        

        stage("Update the Deployment Tags") {
            steps {
                sh """
                   cat deployment-service.yaml
                   sed -i 's/${APP_NAME}.*/${APP_NAME}:${IMAGE_TAG}/g' deployment-service.yaml
                   cat deployment-service.yaml
                """
            }
        }	    

	    
    }
}

