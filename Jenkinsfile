pipeline {
    agent any

    environment {
        MAVEN_HOME = tool 'Maven 3.9.5'
        RECIPIENTS = 'nelsono@inkomo.com'
        DEPLOY_USER = credentials('DEPLOY_USER')   // Jenkins credentials ID for remote user
        DEPLOY_HOST = credentials('DEPLOY_HOST')   // Jenkins credentials ID for remote host
        DEPLOY_PORT = credentials('DEPLOY_PORT')   // Jenkins credentials ID for remote port
        DEPLOY_KEY  = credentials('DEPLOY_KEY')    // Jenkins credentials ID for SSH private key
        DEPLOY_PATH = '/path/to/deploy/'           // Remote path to deploy
    }

    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }
        stage('Build') {
            steps {
                sh "${MAVEN_HOME}/bin/mvn clean package"
            }
        }
        stage('Test') {
            steps {
                sh "${MAVEN_HOME}/bin/mvn test"
            }
        }
        stage('Deploy') {
            steps {
                sh '''
                ARTIFACT=$(ls target/*.jar | head -n 1)
                echo "$DEPLOY_KEY" > key.pem
                chmod 600 key.pem
                scp -i key.pem -P $DEPLOY_PORT -o StrictHostKeyChecking=no $ARTIFACT $DEPLOY_USER@$DEPLOY_HOST:$DEPLOY_PATH
                ssh -i key.pem -p $DEPLOY_PORT -o StrictHostKeyChecking=no $DEPLOY_USER@$DEPLOY_HOST "ls -l $DEPLOY_PATH"
                rm key.pem
                '''
            }
        }
    }

    post {
        success {
            mail to: "${RECIPIENTS}",
                 subject: "Jenkins Build Success: ${env.JOB_NAME} #${env.BUILD_NUMBER}",
                 body: "Good news!\n\nThe build succeeded and was deployed.\n\nJob: ${env.JOB_NAME}\nBuild: ${env.BUILD_NUMBER}\nURL: ${env.BUILD_URL}"
        }
        failure {
            mail to: "${RECIPIENTS}",
                 subject: "Jenkins Build Failed: ${env.JOB_NAME} #${env.BUILD_NUMBER}",
                 body: "Unfortunately, the build or deployment failed.\n\nJob: ${env.JOB_NAME}\nBuild: ${env.BUILD_NUMBER}\nURL: ${env.BUILD_URL}"
        }
    }
}
