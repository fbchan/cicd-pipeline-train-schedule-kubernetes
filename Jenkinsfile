pipeline {
    agent any
    environment {
        //Docker Hub username
        DOCKER_IMAGE_NAME = "foobz/train-schedule"
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
        stage('DeployToProduction - cloud 1') {
            when {
                branch 'master'
            }
            steps {
                //input 'Deploy to Production?'
                milestone(1)
                kubernetesDeploy(
                    kubeconfigId: 'kubeconfig_cloud1',
                    configs: 'train-schedule-kube.yaml',
                    enableConfigSubstitution: true
                )
            }
        }
        stage('DeployToProduction - cloud 2') {
            when {
                branch 'master'
            }
            steps {
                milestone(2)
                kubernetesDeploy(
                    kubeconfigId: 'kubeconfig_cloud2',
                    configs: 'train-schedule-kube.yaml',
                    enableConfigSubstitution: true
                )
            }
        }
    }
}
