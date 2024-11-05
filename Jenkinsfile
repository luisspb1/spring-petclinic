#!/usr/bin/env groovy

pipeline {
    agent { label 'pod-default' }

    stages {
        stage('Check Environment') {
            steps {
                echo '01# Stage - Check Environment'
                echo '(develop y main): Checking environment Java & Maven versions.'
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
            when {
                anyOf {
                    branch 'main'
                    // branch 'develop'
                }
            }
            steps {
                container('maven') {
                    echo '02# Stage - Build'
                    echo '(develop y main): Build a jar file.'
                    sh './mvnw package -Dmaven.test.skip=true'
                }
            }
        }

        stage('Unit Tests') {
            when {
                anyOf {
                    branch 'main'
                    branch 'develop'
                }
            }
            steps {
                container('maven') {
                    echo '03# Stage - Unit Tests'
                    echo '(develop y main): Launch unit tests.'
                    sh 'mvn test'
                    junit '**/target/surefire-reports/*.xml'
                }
            }
        }

        stage('Publish Artifact') {
            when {
                anyOf {
                    branch 'main'
                    branch 'develop'
                }
            }
            steps {
                container('maven') {
                    echo '04# Stage - Deploy Artifact'
                    echo '(develop y main): Deploy artifact to repository.'
                    sh '''
                        mvn -e deploy:deploy-file \
                            -Durl=http://nexus-service:8081/repository/maven-snapshots \
                            -DgroupId=local.moradores \
                            -DartifactId=spring-petclinic \
                            -Dversion=3.3.0-SNAPSHOT \
                            -Dpackaging=jar \
                            -Dfile=target/spring-petclinic-3.3.0-SNAPSHOT.jar
                    '''
                }
            }
        }

        stage('Build & Publish Container Image') {
            when {
                anyOf {
                    branch 'main'
                    branch 'develop'
                }
            }
            steps {
                container('kaniko') {
                    echo '05# Stage - Build & Publish Container Image'
                    echo '(develop y main): Build container image with Kaniko & Publish to container registry.'
                    sh '''
                        /kaniko/executor \
                        --context `pwd` \
                        --insecure \
                        --dockerfile Dockerfile \
                        --destination=nexus-service:8082/repository/docker/spring-petclinic:3.3.0-SNAPSHOT \
                        --destination=nexus-service:8082/repository/docker/spring-petclinic:latest
