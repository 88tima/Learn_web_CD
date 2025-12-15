pipeline {
    agent any
    environment {
        SCRIPT_NAME = 'deployment_case_change.yaml'
    }
    stages {
        stage('Connecting to the server Ubuntu') {
            steps {
                script {
                    withCredentials([
                        string(credentialsId: 'SSH_PORT', variable: 'SSH_PORT'),
                        string(credentialsId: 'IP_FOR_REMOTE', variable: 'IP_FOR_REMOTE'),
                        string(credentialsId: 'REMOTE_USER', variable: 'REMOTE_USER')
                    ]) {
                        sshagent (credentials: ['ubuntu-ssh']) {
                            sh """
                                ssh -o ConnectTimeout=10 -o StrictHostKeyChecking=no \
                                    -p \${SSH_PORT} \
                                    \${REMOTE_USER}@\${IP_FOR_REMOTE} 'echo Connected'
                            """
                        }
                    }
                }
            }
        }

        stage('Start Minikube on remote server') {
            steps {
                script {
                    withCredentials([
                        string(credentialsId: 'SSH_PORT', variable: 'SSH_PORT'),
                        string(credentialsId: 'IP_FOR_REMOTE', variable: 'IP_FOR_REMOTE'),
                        string(credentialsId: 'REMOTE_USER', variable: 'REMOTE_USER')
                    ]) {
                        sshagent (credentials: ['ubuntu-ssh']) {
                            sh """
                                ssh -o StrictHostKeyChecking=no -p \${SSH_PORT} \
                                    \${REMOTE_USER}@\${IP_FOR_REMOTE} '
                                        echo "Starting Minikube..." &&
                                        minikube delete 2>/dev/null || true &&
                                        minikube start --driver=docker
                                    '
                            """
                        }
                    }
                }
            }
        }

        stage('Copy Kubernetes manifest to server') {
            steps {
                script {
                    withCredentials([
                        string(credentialsId: 'SSH_PORT', variable: 'SSH_PORT'),
                        string(credentialsId: 'IP_FOR_REMOTE', variable: 'IP_FOR_REMOTE'),
                        string(credentialsId: 'REMOTE_USER', variable: 'REMOTE_USER')
                    ]) {
                        sshagent (credentials: ['ubuntu-ssh']) {
                            sh """
                                scp -o StrictHostKeyChecking=no -P \${SSH_PORT} \
                                    ${SCRIPT_NAME} \
                                    \${REMOTE_USER}@\${IP_FOR_REMOTE}:/tmp/${SCRIPT_NAME}
                            """
                        }
                    }
                }
            }
        }

        stage('Apply Kubernetes manifest') {
            steps {
                script {
                    withCredentials([
                        string(credentialsId: 'SSH_PORT', variable: 'SSH_PORT'),
                        string(credentialsId: 'IP_FOR_REMOTE', variable: 'IP_FOR_REMOTE'),
                        string(credentialsId: 'REMOTE_USER', variable: 'REMOTE_USER')
                    ]) {
                        sshagent (credentials: ['ubuntu-ssh']) {
                            sh """
                                ssh -o StrictHostKeyChecking=no -p \${SSH_PORT} \
                                    \${REMOTE_USER}@\${IP_FOR_REMOTE} '
                                        export KUBECONFIG=/home/\${USER}/.kube/config &&
                                        kubectl apply -f /tmp/${SCRIPT_NAME}
                                    '
                            """
                        }
                    }
                }
            }
        }
    }

    post {
        always {
            withCredentials([
                string(credentialsId: 'SSH_PORT', variable: 'SSH_PORT'),
                string(credentialsId: 'IP_FOR_REMOTE', variable: 'IP_FOR_REMOTE'),
                string(credentialsId: 'REMOTE_USER', variable: 'REMOTE_USER')
            ]) {
                sshagent (credentials: ['ubuntu-ssh']) {
                    sh """
                        ssh -o StrictHostKeyChecking=no -p \${SSH_PORT} \
                            \${REMOTE_USER}@\${IP_FOR_REMOTE} \
                            'rm -f /tmp/${SCRIPT_NAME}'
                    """
                }
            }
        }
    }
}


