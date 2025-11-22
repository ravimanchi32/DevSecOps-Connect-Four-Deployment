pipeline {
    agent any

    environment {
        IMAGE = "${JOB_NAME}:v1.${BUILD_ID}"
    }

    stages {

        stage('Git Checkout') {
            steps {
                git branch: 'master', url: 'https://github.com/ravimanchi32/DevSecOps-Connect-Four-Deployment.git'
            }
        }

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
                    -Dsonar.login=${SONAR_AUTH_TOKEN}
                """
            }
        }
    }
        }

        stage("Quality Gate") {
            steps {
                script {
                    timeout(time: 2, unit: 'MINUTES') {
                        waitForQualityGate abortPipeline: true
                    }
                }
            }
        }

        stage('Docker Build') {
            steps {
                sh 'docker build -t "$IMAGE" .'
                sh 'docker tag "$IMAGE" ravikumarmanchi/"$JOB_NAME":v1."$BUILD_ID"'
                sh 'docker tag "$IMAGE" ravikumarmanchi/"$JOB_NAME":latest"'
            }
        }

        stage('Docker Login') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'docker', passwordVariable: 'pswd', usernameVariable: 'docker')]) {
                    sh 'docker login -u "${docker}" -p "${pswd}"'
                }
            }
        }

        stage('Docker Push') {
            steps {
                sh 'docker push ravikumarmanchi/"$JOB_NAME":v1."$BUILD_ID"'
                sh 'docker push ravikumarmanchi/"$JOB_NAME":latest"'
            }
        }
    }
}
