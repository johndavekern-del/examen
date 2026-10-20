pipeline {
    agent any

    environment {
        DOCKER_ID    = 'jdk9'
        DOCKER_IMAGE = 'examen-app'
        DOCKER_TAG   = "v1.${BUILD_NUMBER}"
    }

    stages {
        stage('Docker Build') {
            steps {
                sh 'docker build -t $DOCKER_ID/$DOCKER_IMAGE:$DOCKER_TAG .'
            }
        }
        stage('Test Acceptance') {
            steps {
                sh 'docker run -d -p 8081:80 --name test-app $DOCKER_ID/$DOCKER_IMAGE:$DOCKER_TAG && sleep 3 && curl localhost:8081 && docker rm -f test-app'
            }
        }
        stage('Docker Push') {
            steps {
                withCredentials([string(credentialsId: 'DOCKER_HUB_PASS', variable: 'DOCKER_PASS')]) {
                    sh 'echo "$DOCKER_PASS" | docker login -u $DOCKER_ID --password-stdin && docker push $DOCKER_ID/$DOCKER_IMAGE:$DOCKER_TAG'
                }
            }
        }
        stage('Deployment in dev') {
            steps {
                withCredentials([file(credentialsId: 'config', variable: 'KUBECONFIG_FILE')]) {
                    sh 'mkdir -p ~/.kube && cat $KUBECONFIG_FILE > ~/.kube/config && cp fastapi/values.yaml values.yml && sed -i "s+tag.*+tag: ${DOCKER_TAG}+g" values.yml && helm upgrade --install app fastapi --values=values.yml --namespace dev'
                }
            }
        }
        stage('Deployment in staging') {
            steps {
                withCredentials([file(credentialsId: 'config', variable: 'KUBECONFIG_FILE')]) {
                    sh 'mkdir -p ~/.kube && cat $KUBECONFIG_FILE > ~/.kube/config && cp fastapi/values.yaml values.yml && sed -i "s+tag.*+tag: ${DOCKER_TAG}+g" values.yml && helm upgrade --install app fastapi --values=values.yml --namespace staging'
                }
            }
        }
        stage('Deployment in prod') {
            when {
                branch 'main'
            }
            steps {
                timeout(time: 15, unit: 'MINUTES') {
                    input message: 'Möchten Sie in Production deployen?', ok: 'Deploy'
                }
                withCredentials([file(credentialsId: 'config', variable: 'KUBECONFIG_FILE')]) {
                    sh 'mkdir -p ~/.kube && cat $KUBECONFIG_FILE > ~/.kube/config && cp fastapi/values.yaml values.yml && sed -i "s+tag.*+tag: ${DOCKER_TAG}+g" values.yml && helm upgrade --install app fastapi --values=values.yml --namespace prod'
                }
            }
        }
    }
}
