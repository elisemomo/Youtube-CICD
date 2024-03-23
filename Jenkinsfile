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
                    sh "$SCANNER_HOME/bin/sonar-scanner -Dsonar.projectName=reddit-clone -Dsonar.projectKey=reddit-clone"
                }
            }
        } 
}