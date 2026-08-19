pipeline {

    agent any
    tools {
    	jdk 'Java21'
    	maven 'Maven'
        }

    environment {
        DOCKER_IMAGE = "sucharitha929/projectmaven"
        DOCKER_TAG   = "latest"
    }

    stages {

        stage('Checkout') {
            steps {
                git branch: 'main',
                    url: 'https://github.com/Sucharitha929/projectMaven.git',
                    credentialsId: 'github-accesstocken' 
            }
        }

        stage('Build') {
            steps {
                sh 'mvn clean package'
            }
        }

        stage('Test') {
            steps {
                sh 'mvn test'
            }
        }

        stage('Docker Build') {
            steps {
                sh """
                    docker build \
                    -t ${DOCKER_IMAGE}:${DOCKER_TAG} .
                """
            }
        }

        stage('Docker Login') {
            steps {
                withCredentials([
                    usernamePassword(
                        credentialsId: 'dockerhub-credentials',
                        usernameVariable: 'DOCKER_USERNAME',
                        passwordVariable: 'DOCKER_PASSWORD'
                    )
                ]) {

                    sh '''
                        echo "$DOCKER_PASSWORD" | \
                        docker login \
                        -u "$DOCKER_USERNAME" \
                        --password-stdin
                    '''
                }
            }
        }

        stage('Docker Push') {
            steps {
                sh '''
                    docker push ${DOCKER_IMAGE}:${DOCKER_TAG}
                '''
            }
        }

stage('Ansible Deploy') {
    steps {
        withCredentials([
            sshUserPrivateKey(
                credentialsId: 'ansible-access',
                keyFileVariable: 'SSH_KEY',
                usernameVariable: 'SSH_USER'
            )
        ]) {

            sh '''
                mkdir -p ~/.ssh
                chmod 700 ~/.ssh

                ssh-keyscan -H 172.31.33.53 >> ~/.ssh/known_hosts

                chmod 600 "$SSH_KEY"

                ansible-playbook \
                    -i ansible/inventory \
                    ansible/deploy.yml \
                    --private-key "$SSH_KEY" \
                    -u "$SSH_USER"
            '''
        }
    }
}
    }

    post {

        success {
            echo 'Application deployed successfully!'
        }

        failure {
            echo 'Pipeline failed!'
        }
    }
}
