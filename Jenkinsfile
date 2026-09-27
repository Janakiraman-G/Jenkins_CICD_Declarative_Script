pipeline {
    agent any
    
    tools {
        maven 'maven-3'
        jdk 'jdk17'
        dockerTool 'docker'
    }
    
    environment{
        SCANNER_HOME= tool 'sonar-scanner'
    }

    stages {
        stage('Git-Checkout') {
            steps {
                git 'https://github.com/jaiswaladi246/secretsanta-generator.git'
            }
        }
        stage('compile') {
            steps {
                sh "mvn compile"
            }
        }
        stage('test') {
            steps {
                sh "mvn test"
            }
        }
         stage('sonarqube Analysis') {
            steps {
                withSonarQubeEnv('sonarqube') {
                    sh ''' $SCANNER_HOME/bin/sonar-scanner -Dserver.projectName=santa \
                    -Dsonar.projectKey=santa -Dsonar.java.binaries=. '''
                }
            }
        }
        stage('owasp scan') {
            steps {
                dependencyCheck additionalArguments: ' --scan .', odcInstallation: 'DC'
                    dependencyCheckPublisher pattern: '**/dependency-check-report.xml'
            }
        }
        stage('Build application') {
            steps {
                sh 'mvn package'
            }
        }
        stage('Build Docker Image') {
            steps {
                script{
                    withDockerRegistry(credentialsId: 'docker', toolName: 'docker') {
                        sh "docker build -t santa:latest ."
                    }
                }
            }
        }
        stage('Tag & push dokcer image') {
            steps {
                script{
                    withDockerRegistry(credentialsId: 'docker', toolName: 'docker') {
                        sh "docker tag santa:latest jabhuvan/santa:latest"
                        sh "docker push jabhuvan/santa:latest"
                    }
                }
            }
        }
        stage('Deploy Application') {
            steps {
                script{
                    withDockerRegistry(credentialsId: 'docker', toolName: 'docker') {
                        sh "docker run -d -p 8081:8080 jabhuvan/santa:latest"
                    }
                }
            }
        }
    }
}
