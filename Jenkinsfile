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
    
    stage('Trivy Scan') {
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
        stage('Deploy To Nexus') {
            steps {
                script {
                    withMaven(
                        globalMavenSettingsConfig: 'global-settings', 
                        mavenSettingsConfig: 'settings',
                        jdk: 'jdk17', 
                        maven: 'maven3', 
                        traceability: true
                    ) {
                        sh 'mvn deploy -DskipTests=true'
                    }
                }
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

	 stage("Push the changed deployment file to Git") {
            steps {
                sh """
                   git config --global user.name "sami1895"
                   git config --global user.email "badricsami1818@gmail.com"
                   git add deployment-service.yaml
                   git commit -m "Updated Deployment Manifest"
                """
                withCredentials([gitUsernamePassword(credentialsId: 'git-cred', gitToolName: 'Default')]) {
                  sh "git push https://github.com/sami1895/Boardergame main"
            }
        }
        }   

	 stage("slack") {
           steps {
	          slackSend channel: '#jenkins-boarder',
                  color: 'good',
                  failOnError: true,
                  message: "Successful completion of ${env.JOB_NAME} (<${env.BUILD_URL}|Open>)",
                  tokenCredentialId: 'slackboarder'

	    }
        }   
    }
}

