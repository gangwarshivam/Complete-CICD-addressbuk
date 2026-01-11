pipeline{
    agent any
    tools {
        maven 'Maven'
    }
    environment {
        dockerhub_cred = credentials("docker-hub-cred")
        IMAGE_TAG = "${BUILD_NUMBER}"
        IMAGE_NAME = "docker1beginner1shivam/addressbuk"
    }

    stages{
        stage("Checkout"){
            steps{
                checkout scmGit(branches: [[name: '*/master']], extensions: [], userRemoteConfigs: [[url: 'https://github.com/gangwarshivam/addressbook.git']])
            }
        }
        stage("Maven Build"){
            steps{
                sh "mvn clean package"
            }
        }
        stage("SonarQube Analysis") {
            steps{
                script{
                    def mvn = tool 'Maven';
                        withSonarQubeEnv(installationName: 'sonarqube-server') {
                            sh "${mvn}/bin/mvn clean verify org.sonarsource.scanner.maven:sonar-maven-plugin:sonar -Dsonar.projectKey=devops-working -Dsonar.projectName=devops-working"
                        }
                    }
            }
        }
        stage("Docker Build"){
            steps{
                script{
                    sh 'docker build -t $IMAGE_NAME:$IMAGE_TAG .'
                }
            }
        }
        stage("Docker Push"){
            steps{
                script{
                    sh '''
                    echo $dockerhub_cred_PSW | docker login -u $dockerhub_cred_USR --password-stdin
                    docker push $IMAGE_NAME:$IMAGE_TAG
                    docker tag $IMAGE_NAME:$IMAGE_TAG $IMAGE_NAME:latest
                    docker push $IMAGE_NAME:latest
                    '''
                }
            }
        }
        stage("Running a container"){
            steps{
                sh "docker run --name=addressbook -d -p 80:8080 $IMAGE_NAME:latest"
            }
        }
        
    }
    post {
        always {
            echo "Application deployed successfully"
        }
        success {
            echo "It's a success"
        }
        failure {
            echo "Naah, it's a failure"
        }
}

}
