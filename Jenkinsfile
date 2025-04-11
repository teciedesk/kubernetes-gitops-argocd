pipeline {
    agent any

    environment {
        GIT_REPO = 'https://github.com/<your-username>/kubernetes-gitops-argocd.git'
        BRANCH = 'main'
        GIT_CREDENTIALS_ID = 'your-github-creds-id'
    }

    stages {
        stage('Checkout Repo') {
            steps {
                git branch: "${BRANCH}", credentialsId: "${GIT_CREDENTIALS_ID}", url: "${GIT_REPO}"
            }
        }

        stage('YAML Lint') {
            steps {
                echo "Linting ArgoCD Application files..."
                sh 'yamllint argo/*.yaml || true'
            }
        }

        stage('Update App Version (Optional)') {
            when {
                expression { return fileExists('argo/microservices-app.yaml') }
            }
            steps {
                echo "You can use sed/kustomize to update tags here"
                // Example: sh "sed -i 's/tag: .*/tag: 1.0.${BUILD_NUMBER}/' argo/microservices-app.yaml"
            }
        }

        stage('Push Changes') {
            steps {
                sh '''
                git config user.name "Jenkins CI"
                git config user.email "jenkins@ci.local"
                git add argo/*.yaml
                git commit -m "CI/CD: Update ArgoCD App YAMLs - Build #${BUILD_NUMBER}" || echo "No changes to commit"
                git push origin ${BRANCH}
                '''
            }
        }

        stage('Notify') {
            steps {
                echo "Pipeline complete. ArgoCD will detect and sync the changes from Git."
            }
        }
    }
}
