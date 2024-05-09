pipeline {
    agent any
    
    tools {
        jdk 'jdk17'
    }
    
    environment {
        SCANNER_HOME=tool 'sonar-scanner'
    }

    stages {
        stage('Git Checkout') {
            steps {
                git changelog: false, poll: false, url: 'https://github.com/anusha-c-1999/DotNet-DEMO.git'
            }
        }
        
        stage('OWASP Depenency Check ') {
            steps {
                dependencyCheck additionalArguments: ' --scan ./ ', odcInstallation: 'DP-Check'
                dependencyCheckPublisher pattern: '**/dependency-check-report.xml'
            }
        }
        
        stage('Trivy FS') {
            steps {
                sh "trivy fs ."
            }
        }
        stage("Sonarqube Analysis "){
            steps{
                withSonarQubeEnv('sonar-server') {
                    sh ''' $SCANNER_HOME/bin/sonar-scanner -Dsonar.projectName=DotNet-demo \
                    -Dsonar.java.binaries=. \
                    -Dsonar.projectKey=DotNet-demo '''
                }
            }
        }
        
        stage('Docker build and tag') {
            steps {
               script{
                   withDockerRegistry(credentialsId: 'a1b3bde7-d50c-4dc6-875e-29a274222ed4', toolName: 'docker') {
                    sh "make image"
                    }
                }
            }
        } 
        
        stage("Trivy Image Scan "){
            steps{
                sh "trivy image anushac1999/dotnet-demoapp "
            }
        } 
         
        stage('Docker Push') {
            steps {
               script{
                   withDockerRegistry(credentialsId: 'a1b3bde7-d50c-4dc6-875e-29a274222ed4', toolName: 'docker') {
                    sh "make push"
                    }
                }
            }
        }
        
        stage('Deploy to container ') {
            steps {
             script{
                    withDockerRegistry(credentialsId: 'a1b3bde7-d50c-4dc6-875e-29a274222ed4', toolName: 'docker') {
                    sh "docker run -d -p 5000:5000  anushac1999/dotnet-demoapp "
                    }
                }
            }
        }
    }
}
