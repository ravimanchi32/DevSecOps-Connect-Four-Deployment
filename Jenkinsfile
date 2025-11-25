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

        /* ------------------------------
           SONARQUBE SCAN
        --------------------------------*/
        stage('SonarQube Scan') {
            steps {
                script {
                    withSonarQubeEnv('sonar') {   
                        def scannerHome = tool 'sonar-scanner'

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
        // stage('Quality Gate') {
        //     steps {
        //         script {
        //             timeout(time: 5, unit: 'MINUTES') {
        //                 echo "Checking SonarQube Quality Gate..."
        //                 def qg = waitForQualityGate()
        //                 echo "Quality Gate Status: ${qg.status}"
        //                 if (qg.status != 'OK') {
        //                     error "Pipeline failed: Quality Gate status = ${qg.status}"
        //                 }
        //             }
        //         }
        //     }
        // }

        /* ------------------------------
           DOCKER BUILD
        --------------------------------*/
stage('Docker Build & Push') {
    steps {
        script {
            sh '''
                docker build -t ravikumarmanchi/devops:v1.${BUILD_ID} .
                docker tag ravikumarmanchi/devops:v1.${BUILD_ID} ravikumarmanchi/devops:latest
            '''
        }

        withCredentials([usernamePassword(
            credentialsId: 'docker-hub', 
            usernameVariable: 'DOCKER_USER', 
            passwordVariable: 'DOCKER_PASS'
        )]) {
            sh 'echo "$DOCKER_PASS" | docker login -u "$DOCKER_USER" --password-stdin'
        }

        sh '''
            docker push ravikumarmanchi/devops:v1.${BUILD_ID}
            docker push ravikumarmanchi/devops:latest
        '''
    }
}

    }
}
