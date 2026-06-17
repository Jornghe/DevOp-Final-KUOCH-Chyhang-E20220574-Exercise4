pipeline {
    agent any

    triggers {
        pollSCM('H/5 * * * *')
    }

    environment {
        CC_EMAIL          = 'srengty@gmail.com'
        DOCKER            = 'C:\\Program Files\\Docker\\Docker\\resources\\bin\\docker.exe'
        ANSIBLE_INVENTORY = 'D:\\Assignment\\DevOp\\Final_Ex2\\ansible\\inventory.ini'
        ANSIBLE_PLAYBOOK  = 'D:\\Assignment\\DevOp\\Final_Ex2\\ansible\\playbook.yml'
    }

    stages {

        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Build') {
            steps {
                bat '''
                    "%DOCKER%" exec springboot-web bash -c "cd /app && git pull && mvn package -DskipTests -Dmaven.compiler.enablePreview=true"
                '''
            }
        }

        stage('Test') {
            steps {
                bat '''
                    "%DOCKER%" exec -e SPRING_DATASOURCE_URL="jdbc:h2:mem:testdb;DB_CLOSE_DELAY=-1;MODE=MySQL" -e SPRING_DATASOURCE_DRIVER_CLASS_NAME="org.h2.Driver" -e SPRING_DATASOURCE_USERNAME="root" -e SPRING_DATASOURCE_PASSWORD="1234" -e SPRING_JPA_DATABASE_PLATFORM="org.hibernate.dialect.H2Dialect" springboot-web bash -c "cd /app && mvn test -Dspring.profiles.active=test -Dmaven.compiler.enablePreview=true"
                '''
            }
        }

        stage('Deploy with Ansible') {
            steps {
                bat 'wsl ansible-playbook %ANSIBLE_PLAYBOOK% -i %ANSIBLE_INVENTORY%'
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
