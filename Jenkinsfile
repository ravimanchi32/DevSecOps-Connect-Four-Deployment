pipeline {
    agent any

    environment {
        SONAR_TOKEN = credentials('sonar-token')      // Sonar token
        IMAGE = "ravikumarmanchi/devops"              // Docker Image Name
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
        // /* ------------------------------
        //    SONARQUBE SCAN
        // --------------------------------*/
        // stage('SonarQube Scan') {
        //     steps {
        //         script {
        //             try {
        //                 withSonarQubeEnv('sonar') {   
        //                     def scannerHome = tool 'sonar-scanner'

        //                     sh """
        //                         ${scannerHome}/bin/sonar-scanner \
        //                         -Dsonar.projectKey=connect-four \
        //                         -Dsonar.projectName=connect-four \
        //                         -Dsonar.sources=. \
        //                         -Dsonar.host.url=${SONAR_HOST_URL} \
        //                         -Dsonar.login=${SONAR_TOKEN}
        //                     """
        //                 }
        //             } catch (err) {
        //                 error "SonarQube scan failed. Stopping pipeline."
        //             }
        //         }
        //     }
        // }

        // /* ------------------------------
        //    QUALITY GATE
        // --------------------------------*/
        // stage('Quality Gate') {
        //     steps {
        //         script {
        //             timeout(time: 5, unit: 'MINUTES') {
        //                 def qg = waitForQualityGate()
        //                 echo "Quality Gate Status: ${qg.status}"
        //                 if (qg.status != 'OK') {
        //                     error "Pipeline stopped: Quality Gate status = ${qg.status}"
        //                 }
        //             }
        //         }
        //     }
        // }

        /* ------------------------------
           DOCKER BUILD
        --------------------------------*/
        stage('Docker Build') {
            steps {
                sh '''
                    echo "Building Docker image ${IMAGE}:v1.${BUILD_ID}"
                    docker build -t ${IMAGE}:v1.${BUILD_ID} .
                    docker tag ${IMAGE}:v1.${BUILD_ID} ${IMAGE}:latest
                '''
            }
        }

        /* ------------------------------
           TRIVY IMAGE SCAN (Dockerized)
        --------------------------------*/
        stage('Trivy Image Scan') {
            steps {
                sh '''
                    echo "Scanning Docker image ${IMAGE}:v1.${BUILD_ID} with Trivy container..."

                    docker run --rm -v /var/run/docker.sock:/var/run/docker.sock \
                        aquasec/trivy image \
                        --exit-code 1 \
                        --severity HIGH,CRITICAL \
                        --format json \
                        --output trivy-report.json \
                        ${IMAGE}:v1.${BUILD_ID}

                    echo "Trivy scan completed. Report saved as trivy-report.json"
                '''
            }

            post {
                always {
                    archiveArtifacts artifacts: 'trivy-report.json', allowEmptyArchive: true
                }
            }
        }

        /* ------------------------------
           DOCKER LOGIN & PUSH
        --------------------------------*/
        stage('Docker Push') {
            steps {
                withCredentials([usernamePassword(
                    credentialsId: 'docker-hub', 
                    usernameVariable: 'DOCKER_USER', 
                    passwordVariable: 'DOCKER_PASS'
                )]) {
                    sh 'echo "$DOCKER_PASS" | docker login -u "$DOCKER_USER" --password-stdin'
                }

                sh '''
                    docker push ${IMAGE}:v1.${BUILD_ID}
                    docker push ${IMAGE}:latest
                '''
            }
        }

    }

    post {
        failure {
            echo "Pipeline failed. Please check the logs above."
        }
        success {
            echo "Pipeline completed successfully!"
        }
    }
}
