pipeline {
    agent { label 'linux' }

    environment {
        DOCKERHUB_CREDS = credentials('docker hub')
        IMAGE_MR = "krishnav2345/mr"
        IMAGE_MAIN = "krishnav2345/main"
        SHORT_COMMIT = "${env.GIT_COMMIT.take(7)}"
    }

    stages {

        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Checkstyle') {
            when {
                not { branch 'main' }
            }
            steps {
                sh './mvnw checkstyle:checkstyle'
            }
            post {
                always {
                    archiveArtifacts artifacts: 'target/checkstyle-result.xml', fingerprint: true
                }
            }
        }

        stage('Test') {
            when {
                not { branch 'main' }
            }
            steps {
                sh './mvnw test'
            }
            post {
                always {
                    junit 'target/surefire-reports/*.xml'
                }
            }
        }

        stage('Build') {
            when {
                not { branch 'main' }
            }
            steps {
                sh './mvnw clean package -DskipTests'
            }
        }

        stage('Docker Build & Push MR') {
            when {
                not { branch 'main' }
            }
            steps {
                script {
                    def image = docker.build("${IMAGE_MR}:${SHORT_COMMIT}")

                    sh """
                    echo $DOCKERHUB_CREDS_PSW | docker login -u $DOCKERHUB_CREDS_USR --password-stdin
                    docker push ${IMAGE_MR}:${SHORT_COMMIT}
                    """
                }
            }
        }

        stage('Docker Build & Push MAIN') {
            when {
                branch 'main'
            }
            steps {
                sh './mvnw clean package -DskipTests'
                script {
                    def image = docker.build("${IMAGE_MAIN}:${SHORT_COMMIT}")

                    sh """
                    echo $DOCKERHUB_CREDS_PSW | docker login -u $DOCKERHUB_CREDS_USR --password-stdin
                    docker push ${IMAGE_MAIN}:${SHORT_COMMIT}
                    """
                }
            }
        }
    }
}
