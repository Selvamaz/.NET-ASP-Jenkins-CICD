pipeline {
    agent any

    environment {
        DOTNET_CLI_TELEMETRY_OPTOUT = '1'
    }
    
    stages {
        stage('Clone Repository') {
            steps {
                git 'https://github.com/Selvamaz/.NET-ASP-Jenkins-CICD.git'
            }
        }
        stage('Build') {
            steps {
                script {
                    sh 'dotnet restore'
                    sh 'dotnet build --configuration Release'
                }
            }
        }
        stage('Test') {
            steps {
                sh 'dotnet test --no-build --verbosity normal'
            }
        }
        stage('Pack') {
            steps {
                sh 'dotnet publish --configuration Release --output out'
                sh 'zip -r out/hwapp.zip out/*'
            }
        }
        stage('Upload to S3') {
            steps {
                withCredentials([[$class: 'AmazonWebServicesCredentialsBinding', credentialsId: 'AWS User']]) { 
                    sh '''
                    aws s3 cp out/hwapp.zip s3://dotnetarchive/asp/
                    '''
                }
            }
        }
        stage('Docker Build') {
            steps {
                sh 'docker rmi -f hwapp || true'
                sh 'docker build -t hwapp:latest .'
            }
        }
        stage('Deploy') {
            steps {
                script {
                    sh 'docker run -itd -p 8084:80 hwapp:latest'
                }
            }
        }
    }
}
