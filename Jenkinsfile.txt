pipeline {
    agent any
    
    environment {
        SONARQUBE = 'SonarQube' // Name of your SonarQube server in Jenkins
        ARTIFACTORY = 'Artifactory' // Name of your JFrog Artifactory server in Jenkins
        TOMCAT_SERVER = 'tomcat-server' // SSH server for Tomcat deployment
    }
    
    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Build') {
            steps {
                script {
                    sh 'mvn clean install'  // Use Maven or Gradle to build the project
                }
            }
        }

        stage('SonarQube Analysis') {
            steps {
                script {
                    // Run SonarQube analysis
                    sh 'mvn sonar:sonar -Dsonar.projectKey=java-hello-world -Dsonar.host.url=http://<sonarqube-server-url>'
                }
            }
        }

        stage('Push to Artifactory') {
            steps {
                script {
                    // Publish the artifact to Artifactory
                    sh 'mvn deploy -DaltDeploymentRepository=repo::default::http://<artifactory-server-url>/artifactory/libs-release-local'
                }
            }
        }

        stage('Deploy to Tomcat') {
            steps {
                script {
                    // Deploy to Tomcat server
                    sh "scp target/*.war user@${TOMCAT_SERVER}:/path/to/tomcat/webapps/"
                    sh "ssh user@${TOMCAT_SERVER} 'cd /path/to/tomcat/bin && ./shutdown.sh && ./startup.sh'"
                }
            }
        }
    }
    
    post {
        success {
            echo 'Build, Analysis, and Deployment completed successfully!'
        }
        failure {
            echo 'Pipeline failed!'
        }
    }
}
