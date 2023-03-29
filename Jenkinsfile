pipeline {
    agent any
    plugins {
    // id("com.moowork.node") version "1.3.1"
    id("com.github.node-gradle.node") version "2.2.0"
    }
    node {
        download = true
        version = "12.13.1"
        npmVersion = "6.9.0"
        yarnVersion = "1.17.3"
        nodeModulesDir = project.file("ui")
        workDir = project.file("${project.buildDir}/nodejs")
        npmWorkDir = project.file("${project.buildDir}/npm")
        yarnWorkDir = project.file("${project.buildDir}/yarn")
    }
    environment {
        //be sure to replace "willbla" with your own Docker Hub username
        DOCKER_IMAGE_NAME = "ksilvar/train-schedule"
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
                kubernetesDeploy(
                    kubeconfigId: 'kubeconfig',
                    configs: 'train-schedule-kube.yml',
                    enableConfigSubstitution: true
                )
            }
        }
    }
}
