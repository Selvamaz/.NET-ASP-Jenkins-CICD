# .NET-ASP-Jenkins-CICD

Jenkins CICD for ASP.NET Project
<br>

> [!Note]
> **Tools Used : Github, ASP Dotnet SDK 8.0, Jenkins, AWS S3 and Docker**


## Basic
1. Create Ubuntu 24 EC2 Server with SSH, HTTP and your favourite SG ports turned on
2. Install Jenkins, Docker, Git, zip, unzip and awscli
3. Install Dotnet SDK 8.0 from official microsoft linux terminal commands

## Stage 1 : Clone Repository
To retrieve the latest code from the specified GitHub repository
    ```stage('Clone Repository') {
            steps {
                git 'https://github.com/Selvamaz/.NET-ASP-Jenkins-CICD.git'
            }
        }``` <br>

## Stage 2 : Build
To restore dependencies and compile the application into an executable format. 
    ```stage('Build') {
            steps {
                script {
                    sh 'dotnet restore'
                    sh 'dotnet build --configuration Release'
                }
            }
        }``` 
dotnet restore: Downloads and installs any dependencies specified in the project files. This ensures that all required packages are available for the build.
dotnet build --configuration Release: Compiles the application in Release mode, optimizing it for production use. It generates the necessary binaries for the application.

## Stage 3 : Test
This command executes any tests defined in the project. Running tests in the CI/CD pipeline is crucial for maintaining code quality and catching issues early
    ```stage('Test') {
            steps {
                sh 'dotnet test --no-build --verbosity normal'
            }
        }``` 

## Stage 4 : Package
To prepare the application for deployment by publishing the compiled output. 
dotnet publish: Packages the application and its dependencies into a folder for deployment. The -o out option specifies that the output should be placed in the out directory.
    ```stage('Pack') {
            steps {
                sh 'dotnet publish --configuration Release --output out'
                sh 'zip -r out/hwapp.zip out/*'
            }
        }``` 

## Stage 5 : Upload Artifcat to S3
Download AWS Credentials plugin and set up aws credentials in jenkins credentials (AWS User access keys)
Upload out folder to s3 bucket - Saving Artificatory
    ```stage('Upload to S3') {
            steps {
                withCredentials([[$class: 'AmazonWebServicesCredentialsBinding', credentialsId: 'AWS User']]) { 
                    sh '''
                    aws s3 cp out/hwapp.zip s3://dotnetarchive/asp/
                    '''
                }
            }
        }``` 

## Stage 6 : Build Docker Images
To create a Docker image containing the application and its dependencies. Make sure Dockerfile is writted and set it to use out folder which has the artifacts/packages
    ```stage('Docker Build') {
            steps {
                sh 'docker rmi -f hwapp || true'
                sh 'docker build -t hwapp:latest .'
            }
        }``` 

## Stage 7 : Deploy
To run the application in a Docker container
    ```stage('Deploy') {
            steps {
                script {
                    sh 'docker run -itd -p 8084:80 hwapp:latest'
                }
            }
        }``` 

## Additional Works
// Configure the application to listen on port 80
app.Urls.Add("http://0.0.0.0:80");
add above 2 lines in Program.cs
Make sure 0.0.0.0:80 in LaunchSetting.json file

## Run 
Once container will be running in the background. Server will be initiated
