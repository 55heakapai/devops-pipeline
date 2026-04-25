pipeline {
    agent any

    tools {
        jdk 'Java25'
    }

    environment {
        EC2_USER = 'ec2-user'
        EC2_HOST = '3.110.172.106'
        JAR_NAME = 'devops-0.0.1-SNAPSHOT.jar'
        DEPLOY_DIR = '/home/ec2-user/app'
        PEM_FILE = 'F:\\Downloads\\devops-key (1).pem'
    }

    stages {

        stage('Checkout Code') {
            steps {
                git branch: 'main',
                    url: 'https://github.com/55heakapai/devops-pipeline.git'
            }
        }

        stage('Build') {
            steps {
                bat '"G:\\apache-maven-3.9.15\\bin\\mvn.cmd" clean install -DskipTests'
            }
        }

        stage('Archive Artifacts') {
            steps {
                archiveArtifacts artifacts: 'target/*.jar',
                                 fingerprint: true
            }
        }

        stage('Deploy to EC2') {
            steps {
                bat """
                    ssh -i "%PEM_FILE%" -o StrictHostKeyChecking=no %EC2_USER%@%EC2_HOST% "mkdir -p %DEPLOY_DIR%"
                    ssh -i "%PEM_FILE%" -o StrictHostKeyChecking=no %EC2_USER%@%EC2_HOST% "pkill -f %JAR_NAME% || true"
                    scp -i "%PEM_FILE%" -o StrictHostKeyChecking=no target\\%JAR_NAME% %EC2_USER%@%EC2_HOST%:%DEPLOY_DIR%/%JAR_NAME%
                """
            }
        }

        stage('Start Service on EC2') {
            steps {
                bat """
                    ssh -i "%PEM_FILE%" -o StrictHostKeyChecking=no %EC2_USER%@%EC2_HOST% "nohup java -jar %DEPLOY_DIR%/%JAR_NAME% --spring.profiles.active=prod > %DEPLOY_DIR%/app.log 2>&1 &"
                """
            }
        }

    }

    post {
        success {
            echo 'Pipeline SUCCESS! App is live on EC2.'
        }
        failure {
            echo 'Pipeline FAILED. Check logs.'
        }
    }
}
