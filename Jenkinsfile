#!/usr/bin/env groovy

def imageName = 'jenkinsciinfra/account-app'

properties([
    buildDiscarder(logRotator(numToKeepStr: '5', artifactNumToKeepStr: '5')),
    pipelineTriggers([[$class:"SCMTrigger", scmpoll_spec:"H/15 * * * *"]]),
])

node('docker') {
    stage('Build') {
        timestamps {
            deleteDir()
            checkout scm
            docker.image('java:8-alpine').inside('-v /var/run/docker.sock:/var/run/docker.sock') {
                sh './gradlew --no-daemon --info'
                archiveArtifacts artifacts: 'build/libs/*.war', fingerprint: true
            }
        }
    }

    stage('Test') {
        timestamps {
            docker.image('ruby:2.3').inside('-v /var/run/docker.sock:/var/run/docker.sock --group-add=982') {
                sh 'bundle install'
                sh 'rake test'
            }
        }
    }

    /* Assuming we're not inside of a pull request or multibranch pipeline */
    if (!(env.CHANGE_ID || env.BRANCH_NAME)) {
        stage('Publish container') {
            docker.image('java:8-alpine').inside('-v /var/run/docker.sock:/var/run/docker.sock') {
                sh './gradlew --no-daemon --info push'
            }
        }
    }
}
