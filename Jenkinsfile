pipeline {
    agent any
    
    tools {
        jdk 'jdk17'
        maven 'maven3'
    }
    
    environment {
        SCANNER_HOME = tool 'sonar-scanner'
    }

    stages {
        stage('Git Checkout') {
            steps {
                git branch: 'main', changelog: false, poll: false, url: 'https://github.com/SANATASMIA/Ekart-java.git'
            }
        }
        
        stage('Compile & Test') {
            steps {
                sh "mvn clean compile"
            }
        }
        
        stage('OWASP Scan') {
            steps {
                dependencyCheck additionalArguments: '--scan ./', odcInstallation: 'DP'
                dependencyCheckPublisher pattern: '**/dependency-check-report.xml'
            }
        }
    
        stage('SonarQube Analysis') {
            steps {
                withSonarQubeEnv('sonar-server') {
                    sh '''
                        $SCANNER_HOME/bin/sonar-scanner \
                        -Dsonar.projectName=Ekart-java \
                        -Dsonar.java.binaries=. \
                        -Dsonar.projectKey=Ekart-java
                    '''
                }
            }
        }        
      
        stage('Build') {
            steps {
                sh "mvn clean package -DskipTests=true"
            }
        }
     
        stage('Docker-Image') {
            steps {
                script {
                    withDockerRegistry(credentialsId: 'c71c6bea-568f-4d3f-83ae-8225a397540a', toolName: 'docker') {
                        sh "docker build -t ekart-java:latest -f docker/Dockerfile ."
                        sh "docker tag ekart-java:latest sanatasmia400/ekart-java:latest"
                        sh "docker push sanatasmia400/ekart-java:latest"
                    }
                }
            }
        } 
        
        stage('Deploy-Docker') {
            steps {
                script {
                    withDockerRegistry(credentialsId: 'c71c6bea-568f-4d3f-83ae-8225a397540a', toolName: 'docker') {
                        sh "docker run -d -it --name ekart.java -p 8070:8070 ekart-java:latest"

                    }
                }
            }
        }
        
    }
}
