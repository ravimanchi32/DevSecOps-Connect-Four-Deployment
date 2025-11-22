pipeline {
    agent any

    environment {
        IMAGE = "${JOB_NAME}:v1.${BUILD_ID}"
        SONAR_TOKEN = credentials('sonar-token')      // SonarQube token
    }

    stages {

        /* ------------------------------
           GIT CHECKOUT
        --------------------------------*/
        stage('Git Checkout') {
            steps {
                git branch: 'master', url: 'https://github.com/ravimanchi32/DevSecOps-Connect-Four-Deployment.git'
            }
        }

        /* ------------------------------
           SONARQUBE SCAN
        --------------------------------*/
        stage('SonarQube Scan') {
            steps {
                script {
                    withSonarQubeEnv('sonar') {   // sonar = SonarQube name in Jenkins
                        
                        def scannerHome = tool 'sonar-scanner'  // tool name in Jenkins

                        sh """
                            ${scannerHome}/bin/sonar-scanner \
                            -Dsonar.projectKey=connect-four \
                            -Dsonar.projectName=connect-four \
                            -Dsonar.sources=. \
                            -Dsonar.host.url=${SONAR_HOST_URL} \
                            -Dsonar.login=${SONAR_TOKEN}
                        """
                    }
                }
            }
        }

        /* ------------------------------
           QUALITY GATE
        --------------------------------*/
        stage("Quality Gate") {
            steps {
                script {
                    timeout(time: 3, unit: 'MINUTES') {
                        waitForQualityGate abortPipeline: true
                    }
                }
            }
        }

        /* ------------------------------
           DOCKER BUILD
        --------------------------------*/
        stage('Docker Build') {
            steps {
                sh 'docker build -t $IMAGE .'

                sh 'docker tag $IMAGE ravikumarmanchi/$JOB_NAME:v1.$BUILD_ID'

                sh 'docker tag $IMAGE ravikumarmanchi/$JOB_NAME:latest'
            }
        }

        /* ------------------------------
           DOCKER LOGIN
        --------------------------------*/
        stage('Docker Login') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'docker', 
                                                  usernameVariable: 'USER', 
                                                  passwordVariable: 'PASS')]) {
                    sh 'echo "$PASS" | docker login -u "$USER" --password-stdin'
                }
            }
        }

        /* ------------------------------
           DOCKER PUSH
        --------------------------------*/
        stage('Docker Push') {
            steps {
                sh 'docker push ravikumarmanchi/$JOB_NAME:v1.$BUILD_ID'
                sh 'docker push ravikumarmanchi/$JOB_NAME:latest'
            }
        }
    }
}
