pipeline {
    agent any

    tools {
        maven 'maven'
    }
    stages {
        stage('vcs') {
            steps {
                git branch: 'main',  
                    url: 'https://github.com/kishorebittu303/spring-boot-mongo-docker-kkfunda.git'
            }
        }
      //    
        stage('Build') {
            steps {
                sh "mvn clean package"
            }
        }

        // File System Security Scan using Trivy
        stage('File System Trivy Scan') {
            steps {
                script {
                    def status = sh(script: "trivy fs --format table -o trivy-fs-report.html .", returnStatus: true)
                    if (status != 0) {
                        error "Trivy scan failed with exit code ${status}"
                    } else {
                        echo "Trivy scan completed successfully."
                    }
                }
            }
        }
        // manage jenkins-->systemsconfigure-->sonarscannerblock-->name must be given same (eg sonarqube)
        stage('sonar-reports') {
            steps {
                withSonarQubeEnv('sonarqube') {
                    sh """
                    mvn sonar:sonar \
                        -Dsonar.projectKey=spring-boot-mongo \
                        -Dsonar.projectName='Spring Boot Mongo Project' \
                        -Dsonar.host.url=http://54.212.197.72:9000/
                    """
                }
            }
        } 
        // here for our flexibility we create image with our github repo name for pushing easily
       stage('building-image') {
            steps {
                script {
                    withDockerRegistry(credentialsId: 'dockerhub-credentials') {
                        sh "docker build -t bittu3003/mongospring:latest ."
                    }
                }
            }
        }
    //    we are pushing the image
        stage('image_push_dockerhub') {
            steps {
                script {
                    withDockerRegistry(credentialsId: 'dockerhub-credentials') {
                        sh "docker push bittu3003/mongospring:latest"
                    }
                }
            }
        }
        // pushing to tomcat
        // stage('pushing to tomcat'){
        //     steps{
        //         sh """
        //         docker image build -t tomcat:1.0.0 .
        //         docker container run -d -p 8082:8082 tomcat:1.0.0
        //         """
        //     }
        // } it will not work bcoz it is jar file
    } //stages end here
} // pipeline ends here
