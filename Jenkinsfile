def branchPort(String branch) {
        branch == 'dev' ? '3001' : '3000'
}

def branchLogo(String branch) {
        if (branch == 'dev') {
                return '''<svg xmlns="http://www.w3.org/2000/svg" viewBox="0 0 200 200" role="img" aria-label="Dev logo">
    <rect width="200" height="200" rx="24" fill="#0f766e"/>
    <circle cx="100" cy="86" r="42" fill="#ccfbf1"/>
    <path d="M56 142h88l-15-30H71z" fill="#99f6e4"/>
    <text x="100" y="116" fill="#134e4a" font-family="Arial, Helvetica, sans-serif" font-size="36" font-weight="700" text-anchor="middle">DEV</text>
</svg>
'''
        }

        return '''<svg xmlns="http://www.w3.org/2000/svg" viewBox="0 0 200 200" role="img" aria-label="Main logo">
    <rect width="200" height="200" rx="24" fill="#1d4ed8"/>
    <circle cx="100" cy="86" r="42" fill="#dbeafe"/>
    <path d="M56 142h88l-15-30H71z" fill="#93c5fd"/>
    <text x="100" y="116" fill="#1e3a8a" font-family="Arial, Helvetica, sans-serif" font-size="30" font-weight="700" text-anchor="middle">MAIN</text>
</svg>
'''
}

pipeline {
    agent any

    options {
        skipDefaultCheckout(true)
        timestamps()
        disableConcurrentBuilds()
    }

    environment {
        IMAGE_NAME = 'epam-cicd-lab-3'
    }

    stages {
        stage('Checkout') {
            steps {
                checkout scm
                script {
                    env.APP_BRANCH = env.BRANCH_NAME ?: 'main'
                    env.APP_PORT = branchPort(env.APP_BRANCH)
                    env.CONTAINER_NAME = "${env.IMAGE_NAME}-${env.APP_BRANCH}"
                    env.IMAGE_TAG = "${env.IMAGE_NAME}:${env.APP_BRANCH}"
                }
            }
        }

        stage('Build') {
            steps {
                script {
                    writeFile file: 'src/logo.svg', text: branchLogo(env.APP_BRANCH)
                }

                sh '''
                    set -euo pipefail
                    npm install
                    npm run build
                '''
            }
        }

        stage('Test') {
            steps {
                sh 'CI=true npm test -- --runInBand'
            }
        }

        stage('Build docker image') {
            steps {
                sh 'docker build -t "$IMAGE_TAG" .'
            }
        }

        stage('Deploy') {
            steps {
                sh '''
                    set -euo pipefail

                    docker rm -f "${CONTAINER_NAME}" >/dev/null 2>&1 || true
                    docker run -d \
                        --name "${CONTAINER_NAME}" \
                        -e HOST=0.0.0.0 \
                        -e PORT="${APP_PORT}" \
                        -p "${APP_PORT}:${APP_PORT}" \
                        "${IMAGE_TAG}"
                '''
            }
        }
    }
}