pipeline {
    agent any

    environment {
        DOCKER_IMAGE = "yourdockerhubusername/socialmedia:latest"
    }

    stages {
        stage('Checkout') {
            steps {
                git 'https://github.com/yourusername/socialmedia.git'
            }
        }

        stage('Build') {
            steps {
                sh 'mvn clean package' // For Java
                // sh 'npm install && npm run build' // For JavaScript
            }
        }

        stage('Test') {
            steps {
                sh 'mvn test' // Java
                // sh 'npm test' // JavaScript
            }
        }

        stage('Code Quality Check') {
            steps {
                script {
                    // Run SonarQube or other code quality analysis
                    sh 'sonar-scanner -Dsonar.projectKey=socialmedia'
                }
            }
        }

        stage('Build Docker Image') {
            steps {
                script {
                    sh "docker build -t ${DOCKER_IMAGE} ."
                }
            }
        }

        stage('Push Docker Image') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'docker-hub-creds', usernameVariable: 'DOCKER_USER', passwordVariable: 'DOCKER_PASS')]) {
                    script {
                        sh "echo $DOCKER_PASS | docker login -u $DOCKER_USER --password-stdin"
                        sh "docker push ${DOCKER_IMAGE}"
                    }
                }
            }
        }

        stage('Deploy') {
            steps {
                sshagent(['deployment-server-creds']) {
                    sh 'ssh user@yourdeploymentserver.com "docker pull ${DOCKER_IMAGE} && docker run -d -p 80:80 ${DOCKER_IMAGE}"'
                }
            }
        }
    }

    post {
        always {
            mail to: 'team@example.com',
                 subject: "Jenkins Build ${currentBuild.fullDisplayName}",
                 body: "Build ${currentBuild.currentResult}. Check console output at ${env.BUILD_URL}"
        }
    }
}
