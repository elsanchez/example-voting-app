pipeline {

    agent none


    stages {
        stage('worker-build') {
            agent {
                docker {
                    image 'maven:3.6.1-jdk-8-alpine'
                    args '-v $HOME/.m2:/root/.m2'
                }
            }
            when {
                changeset "**/worker/**"
            }
            steps {
                echo 'Compiling worker app'
                dir('worker') {
                    sh 'mvn compile'
                }
            }
        }
        stage('worker-test') {

            agent {
                docker {
                    image 'maven:3.6.1-jdk-8-alpine'
                    args '-v $HOME/.m2:/root/.m2'
                }
            }

            when {
                changeset "**/worker/**"
            }
            steps {
                echo 'Running Unit Tests on worker app'
                dir('worker') {
                    sh 'mvn clean test'
                }
            }
        }
        stage('worker-package') {

            agent {
                docker {
                    image 'maven:3.6.1-jdk-8-alpine'
                    args '-v $HOME/.m2:/root/.m2'
                }
            }
            when {
                branch 'master'
                changeset "**/worker/**"
            }
            steps {
                echo 'Packaging woker app'
                dir('worker') {
                    sh 'mvn package -D skipTests'
                    archiveArtifacts artifacts: '**/target/*.jar', fingerprint: true
                }
            }
        }

        stage('worker-docker-package') {

            agent any

            when {
                //branch 'master'
                changeset "**/worker/**"
            }
            steps {
                echo 'Packaging woker app with docker'
                script {
                    docker.withRegistry('https://index.docker.io/v1/', 'dockerlogin') {
                        def workerImage = docker.build("elsanchez/worker:v${env.BUILD_ID}", "./worker")
                        workerImage.push()
                        workerImage.push("latest")
                    }
                }
            }
        }

        stage('result-build') {
            agent {
                docker {
                    image 'node:8.16.0-alpine'
                }
            }
            when {
                changeset "**/result/**"
            }
            steps {
                echo 'Bulding install'
                dir('result') {
                    sh 'npm install'
                }
            }
        }
        stage('result-test') {
            agent {
                docker {
                    image 'node:8.16.0-alpine'
                }
            }
            when {
                changeset "**/result/**"
            }
            steps {
                echo 'Running test on result app'
                dir('result') {
                    sh 'npm install'
                    sh 'npm test'
                }
            }
        }

        stage('result-docker-package') {

            agent any

            when {
                //branch 'master'
                changeset "**/result/**"
            }
            steps {
                echo 'Packaging result app with docker'
                script {
                    docker.withRegistry('https://index.docker.io/v1/', 'dockerlogin') {
                        def resultImage = docker.build("elsanchez/result:v${env.BUILD_ID}", "./result")
                        resultImage.push()
                        resultImage.push("latest")
                    }
                }
            }
        }

        stage('vote-build') {
            agent {
                docker {
                    image 'python:2.7.16-slim'
                    args '--user root'
                }
            }
            when {
                changeset "**/vote/**"
            }
            steps {
                echo 'Bulding install'
                dir('vote') {
                    sh 'pip install -r requirements.txt'
                }
            }
        }
        stage('vote-test') {
            agent {
                docker {
                    image 'python:2.7.16-slim'
                    args '--user root'
                }
            }
            when {
                changeset "**/vote/**"
            }
            steps {
                echo 'Running test on vote app'
                dir('vote') {
                    sh 'pip install -r requirements.txt'
                    sh 'nosetests -v'
                }
            }
        }

        stage('vote-docker-package') {

            agent any

            when {
                //branch 'master'
                changeset "**/vote/**"
            }
            steps {
                echo 'Packaging vote app with docker'
                script {
                    docker.withRegistry('https://index.docker.io/v1/', 'dockerlogin') {
                        def voteImage = docker.build("elsanchez/vote:v${env.BUILD_ID}", "./vote")
                        voteImage.push()
                        voteImage.push("latest")
                    }
                }
            }
        }
    }

    post {
        failure {
            slackSend (channel: "instavote-cd", message: "Build Failed - ${env.JOB_NAME} ${env.BUILD_NUMBER} (<${env.BUILD_URL}|Open>)")
        }
        success {
            slackSend (channel: "instavote-cd", message: "Build Succeeded - ${env.JOB_NAME} ${env.BUILD_NUMBER} (<${env.BUILD_URL}|Open>)")
        }
        always {
            echo 'Build pipeline is complete'
        }
    }
}