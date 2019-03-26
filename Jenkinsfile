pipeline {
    agent any
    environment {
        //Docker Hub username
        APPS_NAME = "train"
        FQDN = "train.foobz.com.au"
        DOCKER_IMAGE_NAME = "foobz/train-schedule"
    }
    stages {
        stage('Build Apps and Test') {
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
        stage('Container Security Scan') {
            steps {
                echo 'Scanning container image for vulnerability ....'
                sh 'echo "${DOCKER_IMAGE_NAME} `pwd`/Dockerfile" > anchore_images'
                anchore name: 'anchore_images'
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
        stage('Cleanup') {
            steps {
                sh'''
                    for i in `cat anchore_images | awk '{print $1}'`;do docker rmi $i; done
                '''
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
        stage('DeployAppServices - AS3') {
            steps {
                // Deploy AppServices with AS3
                milestone(3)
                build (job: "bigip-app-services-as3-SD", 
                       parameters: 
                       [string(name: 'FQDN', value: FQDN),
                       string(name: 'APPS_NAME', value: APPS_NAME)])
            }
        }
    }
}
