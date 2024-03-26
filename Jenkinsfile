pipeline {
    agent any
    tools {

        nodejs 'NodeJs'
    }
    
    environment {
        SCANNER_HOME = tool 'sonar-scanner'
        DOCKERHUB_CREDENTIALS = credentials('Docker_hub')
        GITHUB_REPO_URL = 'https://github.com/elisemomo/Youtube-CICD.git'
    }
    
    stages {
        stage('Clean workspace') {
            steps {
                cleanWs()
            }
        }
        
        stage('Checkout') {
            steps {
                git branch: 'main', url: 'https://github.com/elisemomo/Youtube-CICD.git'
            }
        }
        
        
        stage('NPM') {
            steps {
                sh 'npm install'
            }
        }
        
        stage('SonarQube Analysis') {
            steps {
                withSonarQubeEnv('sonar-server') {
                    sh "$SCANNER_HOME/bin/sonar-scanner -Dsonar.projectName=youtube -Dsonar.projectKey=youtube"
                }
            }
        } 

        stage('Quality Gate') {
            steps {
                script {
                    waitForQualityGate abortPipeline: false, credentialsId: 'Sonar-Token'
                }
            }
        }
        
        stage('Trivy File Scan') {
            steps {
                script {
                    sh '/usr/local/bin/trivy fs . > trivy_result.txt'
                }
            }
        }

        stage('Dependency-Check') {
            steps {
                dependencyCheck additionalArguments: '', nvdCredentialsId: 'nvdAPIkey', odcInstallation: 'Dependency-Check'
                dependencyCheckPublisher pattern: '**/dependency-check-report.xml'
            }
        }
        stage('Login to DockerHUB') {
            steps {
                script {
                    sh "echo $DOCKERHUB_CREDENTIALS_PSW | docker login -u $DOCKERHUB_CREDENTIALS_USR --password-stdin"
                    echo 'Login Succeeded'
                }
            }
        }

        stage('Docker Build') {
            steps {
                script {
                    sh 'docker build -t mukomelise/youtube:latest .' 
                    echo "Image Build Successfully"
                }
            }
        }

        
        stage('Docker Push') {
            steps {
                script {
                    sh 'docker push mukomelise/youtube:latest'
                    echo "Push Image to Registry"
                }
            }
        }

        stage('Trivy Image Scan') {
            steps {
                script {
                    sh '/usr/local/bin/trivy image mukomelise/youtube:latest > trivy_image_result.txt'
                    sh 'pwd'
                }
            }
        }

        stage('RUN CONTAINER') {
            steps {
                script {
                    def containername = 'youtube-clone'
                    def isRunning = sh(script: "docker ps -a | grep ${containername}", returnStatus: true)
                    if (isRunning == 0) {
                        sh "docker stop ${containername}"
                        sh "docker rm ${containername}"
                    }
                    sh "docker run -d -p 3000:3000 --name ${containername} mukomelise/youtube:latest"
                }
                    
              
            }
        }
        
        stage('Deploy to Kubernetes') {
            steps {
                withKubeConfig([credentialsId: 'kubeconfig']) {
                    sh 'kubectl apply -f Kubernetes'
                }
            }
      }

      stage('slack notification'){
        steps {
            script{
                slackSend channel: '#cicd-projects', message: 'Build Success', teamDomain: 'elisemomo', tokenCredentialId: 'slack-token'
            }
            
        }
      }
  }
}