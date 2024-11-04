#!/usr/bin/env groovy
pipeline {
    agent { label 'pod-default' }
    stages {
        stage('Check Environment') {
            steps {
                println '01# Stage - Check Environment'
                println '(develop y main):  Checking environment Java & Maven versions.'
                sh 'java -version'
                container('maven') {
                    sh '''
                        java -version
                        mvn -version
                        pwd
                    '''
                }
            }
        }
        stage('Build') {
            steps {
                container('maven') {
                    println '02# Stage - Build'
                    println '(develop y main):  Build a jar file.'
                    sh './mvnw package -Dmaven.test.skip=true'
                    archiveArtifacts artifacts: 'target/*.jar', fingerprint: true
                }
            }
        }
    }
}