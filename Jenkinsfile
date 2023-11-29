pipeline {
    agent any
    environment {
        DOCKER_IMAGE_NAME = "hahashem/train-schedule"
    }
    stages {
        stage('Build') {
            steps {
                echo 'Running build automation'
                sh './gradlew build --no-daemon'
                archiveArtifacts artifacts: 'dist/trainSchedule.zip'
            }
        }
        stage('Build Docker Image') {
            when {
                branch 'master'
            }
            steps {
                script {
                    app = docker.build(DOCKER_IMAGE_NAME)
                    app.inside {
                        sh 'echo Hello, World!'
                    }
                }
            }
        }
        stage('Push Docker Image') {
            when {
                branch 'master'
            }
            steps {
                script {
                    docker.withRegistry('https://registry.hub.docker.com', 'docker_hub_login') {
                        app.push("${env.BUILD_NUMBER}")
                        app.push("latest")
                    }
                }
            }
        }
        stage('DeployToProduction') {
            when {
                branch 'master'
            }
            steps {
                input 'Deploy to Production?'
                milestone(1)
                withCredentials([usernamePassword(credentialsId: 'webserver_login', usernameVariable: 'USERNAME', passwordVariable: 'USERPASS')]) {
                	script {
                		sh "envsubst < ./train-schedule-kube.yml > /tmp/train-schedule-kube.yml && sshpass -p '$USERPASS' -v scp /tmp/train-schedule-kube.yml $USERNAME@$prod_ip:/tmp/ && rm /tmp/train-schedule-kube.yml"
                		sh "sshpass -p '$USERPASS' -v ssh -o StrictHostKeyChecking=no $USERNAME@$prod_ip \"kubectl apply -f /tmp/train-schedule-kube.yml && rm /tmp/train-schedule-kube.yml\""
                	}
                }
            }
        }
    }
}
