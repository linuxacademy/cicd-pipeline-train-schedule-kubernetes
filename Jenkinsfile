agentName = "server-slave"

pipeline {
    agent none
    environment {
        //be sure to replace "willbla" with your own Docker Hub username
        MYIMAGE = "$DOCKER_IMAGE_NAME:$BUILD_NUMBER"
        DOCKER_IMAGE_NAME = "shdh/train-schedule"
        
    }
    stages {
        stage('Build') {
            agent { label agentName }
            steps {
                echo 'Running build automation'
                sh './gradlew build --no-daemon'
                archiveArtifacts artifacts: 'dist/trainSchedule.zip'
            }
        }
        stage('Build Docker Image') {
            when {
                branch 'development'
            }
            agent { label agentName }
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
                branch 'development'
            }
            agent { label agentName }
            steps {
                script {
                    docker.withRegistry('https://registry.hub.docker.com', 'docker_hub_login') {
                        app.push("${env.BUILD_NUMBER}")
                        app.push("latest")
                    }
                }
            }
        }
        stage('DeployToDev') {
            when {
                branch 'development'
            }
            agent { label agentName }
            environment {
              NAMESPACE = "dev"  
              NODEPORT = "30050" 
           }
            steps {
                kubernetesDeploy(
                    kubeconfigId: 'kube',
                    configs: 'train-schedule-kube.yml',
                    enableConfigSubstitution: true
                )
            }
        }
        stage('DeployToCT') {
            when {
                branch 'development'
            }
            agent { label agentName }
            environment {
              NAMESPACE = "ct"  
              NODEPORT = "30060"
           }
            steps {
                input 'Deploy to CT?'
                milestone(1)
                kubernetesDeploy(
                    kubeconfigId: 'kube',
                    configs: 'train-schedule-kube.yml',
                    enableConfigSubstitution: true
                )
            }
        }
        stage('DeployToProduction') {
            when {
                branch 'master'
            }
            agent { label agentName }
            environment {
              NAMESPACE = "prod"  
              NODEPORT = "30070"
           }
            steps {
                input 'Deploy to Production?'
                milestone(1)
                kubernetesDeploy(
                    kubeconfigId: 'kube',
                    configs: 'train-schedule-kube.yml',
                    enableConfigSubstitution: true
                )
            }
        }
    }
}
