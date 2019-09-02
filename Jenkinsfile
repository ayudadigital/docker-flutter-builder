#!groovy

@Library('github.com/red-panda-ci/jenkins-pipeline-library@v3.1.6') _

// Initialize global config
cfg = jplConfig('docker-flutter-builder', 'docker', '', [slack: '', email:'pedroamador.rodriguez+teecke@gmail.com'])
String jenkinsVersion

def publishDockerImage(String jenkinsVersion) {
    docker.withRegistry("https://registry.hub.docker.com", 'teeckebot-docker-credentials') {
        script (
            sh '''for tag in $(cat build-tools.list); do
                docker push teecke/docker-flutter-builder:${tag}
            done
            '''
        )
    }
}

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
            steps {
                script {
                    sh "devcontrol build-all"
                }
            }
        }
        stage('Make release') {
            when { expression { cfg.BRANCH_NAME.startsWith('release/new') } }
            steps {
                publishDockerImage(jenkinsVersion)
                jplMakeRelease(cfg, true)
                jplCloseRelease(cfg)
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
        timeout(time: 30, unit: 'MINUTES')
    }
}