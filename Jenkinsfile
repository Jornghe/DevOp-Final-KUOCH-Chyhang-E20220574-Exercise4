pipeline {
    agent any

    triggers {
        pollSCM('H/5 * * * *')
    }

    environment {
        CC_EMAIL          = 'srengty@gmail.com'
        ANSIBLE_INVENTORY = 'D:/Assignment/DevOp/Final_Ex2/ansible/inventory.ini'
        ANSIBLE_PLAYBOOK  = 'D:/Assignment/DevOp/Final_Ex2/ansible/playbook.yml'
    }

    stages {

        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Build') {
            steps {
                sh '''
                    docker exec springboot-web bash -c "
                        cd /app &&
                        git pull &&
                        mvn package -DskipTests -Dmaven.compiler.enablePreview=true
                    "
                '''
            }
        }

        stage('Test') {
            steps {
                sh '''
                    docker exec springboot-web bash -c "
                        cd /app &&
                        mvn test -Dspring.profiles.active=test -Dmaven.compiler.enablePreview=true
                    "
                '''
            }
        }

        stage('Deploy with Ansible') {
            steps {
                sh 'wsl ansible-playbook ${ANSIBLE_PLAYBOOK} -i ${ANSIBLE_INVENTORY}'
            }
        }
    }

    post {
        failure {
            emailext(
                to: "${CC_EMAIL}",
                subject: "BUILD FAILED: ${env.JOB_NAME} #${env.BUILD_NUMBER}",
                body: """
Build failed for job: ${env.JOB_NAME}
Build number: ${env.BUILD_NUMBER}
Committed by: ${env.GIT_AUTHOR_NAME} <${env.GIT_AUTHOR_EMAIL}>
Build URL: ${env.BUILD_URL}

Please check the console output for details.
""",
                replyTo: '',
                mimeType: 'text/plain'
            )
        }
        success {
            echo "Build, test, and deployment completed successfully."
        }
    }
}
