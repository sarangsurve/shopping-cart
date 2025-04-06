pipeline {
    agent any
    tools{
        jdk 'jdk21'
        maven 'maven3'
    }
    
    environment {
        SCANNER_HOME = tool 'sonar-scanner'
    }

    stages {
        stage('Git Checkout') {
            steps {
                git branch: 'main', changelog: false, credentialsId: '3a58dd3d-fb11-4de7-bde5-ec22ce796a37', poll: false, url: 'https://github.com/sarangsurve/shopping-cart.git'
            }
        }
        
        stage('Compile') {
            steps {
                sh "mvn clean compile -DskipTests=true"
            }
        }
        
        // stage('OWASP Scan'){
        //     steps {
        //         dependencyCheck additionalArguments: '--scan ./ ', odcInstallation: 'DP'
        //         dependencyCheckPublisher pattern: '**/dependency-check-report.xml'
        //     }
        // }
        
        stage('Sonarqube') {
            steps {
                withSonarQubeEnv('sonar-server') {
                    sh ''' $SCANNER_HOME/bin/sonar-scanner -Dsonar.projectName=shopping-cart \
                    -Dsonar.java.binaries=. \
                    -Dsonar.projectKey=shopping-cart '''
                }
            }
        }
        
        stage('Build') {
            steps {
                sh "mvn clean package -DskipTests=true"
            }
        }
        
        stage('Docker Build and Push') {
            steps {
                script{
                    withDockerRegistry(credentialsId: 'fac3c25b-e9c9-41c0-90b1-a33fccd59c04', toolName: 'docker') {
                        sh "docker build -t shopping-cart -f docker/Dockerfile ."
                        sh "docker tag shopping-cart sarangsurve/shopping-cart:latest"
                        sh "docker push sarangsurve/shopping-cart:latest"
                    }
                }
            }
        }
    }
}
