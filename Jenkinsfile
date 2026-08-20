pipeline {
    agent { label 'Jenkins-agent' }

    parameters {
        string(
            name: 'IMAGE_TAG',
            defaultValue: 'latest',
            description: 'Docker image tag to deploy'
        )
    }

    options {
        skipDefaultCheckout(true)
    }

    stages {

        stage("Cleanup Workspace") {
            steps {
                cleanWs()
            }
        }

        stage("Checkout from GitOps Repository") {
            steps {
                git branch: 'main',
                    credentialsId: 'github',
                    url: 'https://github.com/rapurusirisha/gitops-register-app.git'
            }
        }

        stage("Update the Deployment Tags") {
            steps {
                sh '''
                    echo "Current deployment.yaml:"
                    cat deployment.yaml

                    echo "Updating image tag to ${IMAGE_TAG}..."

                    sed -i "s|rapurusirisha/register-app-pipeline:.*|rapurusirisha/register-app-pipeline:${IMAGE_TAG}|" deployment.yaml

                    echo "Updated deployment.yaml:"
                    cat deployment.yaml
                '''
            }
        }

        stage("Push the changed deployment file to Git") {
            steps {
                withCredentials([
                    usernamePassword(
                        credentialsId: 'github',
                        usernameVariable: 'GIT_USERNAME',
                        passwordVariable: 'GIT_PASSWORD'
                    )
                ]) {
                    sh '''
                        git config user.name "Jenkins"
                        git config user.email "jenkins@example.com"

                        git add deployment.yaml

                        git diff --cached --quiet || \
                        git commit -m "Update image tag to ${IMAGE_TAG}"

                        git push https://${GIT_USERNAME}:${GIT_PASSWORD}@github.com/rapurusirisha/gitops-register-app.git HEAD:main
                    '''
                }
            }
        }
    }

    post {
        success {
            echo "========================================="
            echo "CD Pipeline completed successfully!"
            echo "Image Tag: ${IMAGE_TAG}"
            echo "GitOps repository updated successfully."
            echo "========================================="
        }

        failure {
            echo "CD Pipeline failed."
        }
    }
}
