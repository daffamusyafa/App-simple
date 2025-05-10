def server = 'finaltask-daffa@147.139.182.172'
def directory = 'test/simple-java-maven-app/quickstart-tutorials'
def branch = 'main'
def image = 'ghcr.io/jenkins-docs/quickstart-tutorials/jenkinsci-tutorials:simple_controller_'

pipeline {
    agent any

    environment {
        secret = 'global'
        discordWebhook = credentials('discord-webhook')
    }

    stages {
        stage ('Pulling New Code') {
            steps {
                sshagent([env.secret]) {
                    sh """ssh -p 1234 -o StrictHostKeyChecking=no ${server} << EOF
                    cd ${directory}
                    git pull origin ${branch}
                    exit
                    EOF"""
                }
                sh """
                curl -X POST -H "Content-Type: application/json" \
                -d '{"content": "✅ Stage: Pulling New Code berhasil."}' \
                "${env.discordWebhook}"
                """
            }
        }

        stage ('Build') {
            steps {
                sshagent([env.secret]) {
                    sh """ssh -p 1234 -o StrictHostKeyChecking=no ${server} << EOF
                    cd ${directory}
                    docker build --no-cache -t ${image} .
                    exit
                    EOF"""
                }
                sh """
                curl -X POST -H "Content-Type: application/json" \
                -d '{"content": "✅ Stage: Build Image berhasil."}' \
                "${env.discordWebhook}"
                """
            }
        }

        stage ('Testing Trivy') {
            steps {
                sshagent([env.secret]) {
                    sh """ssh -p 1234 -o StrictHostKeyChecking=no ${server} << EOF
                    cd ${directory}
                    trivy image --severity CRITICAL,HIGH ${image}
                    echo "Trivy Scan Success"
                    exit
                    EOF"""
                }
                sh """
                curl -X POST -H "Content-Type: application/json" \
                -d '{"content": "✅ Stage: Testing Trivy berhasil."}' \
                "${env.discordWebhook}"
                """
            }
        }

        stage ('Push') {
            steps {
                sshagent([env.secret]) {
                    sh """ssh -p 1234 -o StrictHostKeyChecking=no ${server} << EOF
                    cd ${directory}
                    docker push ${image}
                    exit
                    EOF"""
                }
                sh """
                curl -X POST -H "Content-Type: application/json" \
                -d '{"content": "✅ Stage: Push berhasil."}' \
                "${env.discordWebhook}"
                """
            }
        }

        stage ('Deploy') {
            steps {
                sshagent([env.secret]) {
                    sh """ssh -p 1234 -o StrictHostKeyChecking=no ${server} << EOF
                    cd ${directory}
                    docker compose pull
                    docker compose down
                    docker compose up -d
                    exit
                    EOF"""
                }
                sh """
                curl -X POST -H "Content-Type: application/json" \
                -d '{"content": "✅ Stage: Deploy berhasil. Aplikasi berhasil di-deploy!"}' \
                "${env.discordWebhook}"
                """
            }
        }
    }

    post {
        failure {
            sh """
            curl -X POST -H "Content-Type: application/json" \
            -d '{"content": "❌ Pipeline gagal. Mohon periksa logs Jenkins untuk informasi lebih lanjut."}' \
            "${env.discordWebhook}"
            """
        }
    }
}
