pipeline {
    agent any
    
    tools {
        maven 'Maven 3.9'
        jdk 'JDK 17'
    }
    
    environment {
        DOCKERHUB_CREDENTIALS = credentials('docker-hub-creds')
        RENDER_DEPLOY_HOOK = credentials('render-deploy-hook')
        DOCKER_IMAGE = 'aymanwageah/my-app'
    }
    
    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }
        
        stage('Build with Maven') {
            steps {
                sh 'mvn clean package'
            }
        }
        
        stage('Archive Artifacts') {
            steps {
                archiveArtifacts artifacts: 'target/*.jar', fingerprint: true
            }
        }
        
        stage('Build Docker Image') {
            steps {
                script {
                    docker.build("${DOCKER_IMAGE}:latest")
                }
            }
        }
        
        stage('Push to Docker Hub') {
            when {
                branch 'main'
            }
            steps {
                script {
                    docker.withRegistry('https://registry.hub.docker.com', 'docker-hub-creds') {
                        docker.image("${DOCKER_IMAGE}:latest").push()
                        docker.image("${DOCKER_IMAGE}:latest").push("${env.BUILD_NUMBER}")
                    }
                }
            }
        }
        
        stage('Trigger Render Deployment') {
            when {
                branch 'main'
            }
            steps {
                sh '''
                    curl -X POST ${RENDER_DEPLOY_HOOK}
                '''
            }
        }
    }
    
    post {
        always {
            cleanWs()
        }
        success {
            echo 'Pipeline completed successfully!'
        }
        failure {
            echo 'Pipeline failed!'
        }
    }
}