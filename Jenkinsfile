#!groovy

@Library('github.com/ayudadigital/jenkins-pipeline-library@v4.0.0') _

// Initialize global config
cfg = jplConfig('docker-flutter-builder', 'docker', '', [email: env.CI_NOTIFY_EMAIL_TARGETS])

pipeline {
    agent { label 'docker' }

    stages {
        stage ('Initialize') {
            steps  {
                jplStart(cfg)
            }
        }
        stage ('Bash linter') {
            steps {
                script {
                    sh "devcontrol run-bash-linter"
                }
            }
        }
        stage ('Build') {
            when { branch 'release/new' }
            steps {
                script {
                    sh "devcontrol build-all"
                }
            }
        }
        stage('Make release') {
            when { branch 'release/new' }
            steps {
                // Push all images with one sentence without using jenkins docker integration
                withCredentials([[$class: 'UsernamePasswordMultiBinding', credentialsId: 'docker-token', usernameVariable: 'USERNAME', passwordVariable: 'PASSWORD']]) {
                    sh '''docker login -u ${USERNAME} -p ${PASSWORD}
                    docker push teecke/docker-flutter-builder
                    '''
                }
                jplMakeRelease(cfg, true)
            }
            post {
                always {
                    sh 'docker logout'
                }
            }
        }
    }

    post {
        always {
            jplPostBuild(cfg)
        }
    }

    options {
        timestamps()
        ansiColor('xterm')
        buildDiscarder(logRotator(artifactNumToKeepStr: '20',artifactDaysToKeepStr: '30'))
        disableConcurrentBuilds()
        timeout(time: 180, unit: 'MINUTES')
    }
}
