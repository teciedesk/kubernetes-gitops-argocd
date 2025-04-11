pipeline {
    agent any

    environment {
        REPO_URL = 'https://github.com/teciedesk/kubernetes-gitops-argocd.git'
    }

    stages {
        stage('Checkout Repo') {
            steps {
                git url: "${REPO_URL}", branch: 'main'
            }
        }

        stage('YAML Lint') {
            steps {
                echo 'Linting ArgoCD Application files...'
                sh '''
                    if command -v yamllint >/dev/null 2>&1; then
                        yamllint argo/*.yaml
                    else
                        echo "yamllint not found, skipping YAML linting"
                    fi
                '''
            }
        }

        stage('Update App Version (Optional)') {
            steps {
                echo 'You can use sed/kustomize to update tags here'
                // Optional step for future automation
            }
        }

        stage('Push Changes') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'github-creds', usernameVariable: 'GIT_USER', passwordVariable: 'GIT_PASS')]) {
                    sh '''
                        git config user.name "Jenkins CI"
                        git config user.email jenkins@ci.local
                        git add argo/*.yaml
                        git diff --cached --quiet || git commit -m "CI/CD: Update ArgoCD App YAMLs - Build #${BUILD_NUMBER}"
                        
                        if git diff --cached --quiet; then
                            echo "No changes to commit"
                        else
                            git remote set-url origin https://${GIT_USER}:${GIT_PASS}@github.com/teciedesk/kubernetes-gitops-argocd.git
                            git push origin main
                        fi
                    '''
                }
            }
        }

        stage('Notify') {
            when {
                expression { currentBuild.resultIsBetterOrEqualTo('SUCCESS') }
            }
            steps {
                echo 'Pipeline completed successfully!'
            }
        }
    }
}
