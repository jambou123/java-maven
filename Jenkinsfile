#!/usr/bin/env groovy

pipeline {
    agent any
    
    tools {
        maven 'LocalMaven'
    }
    
    stages {
        stage('increment version') {
            steps {
                script {
                    echo 'incrementing app version...'
                    sh 'mvn build-helper:parse-version versions:set \
                        -DnewVersion=\\\${parsedVersion.majorVersion}.\\\${parsedVersion.minorVersion}.\\\${parsedVersion.nextIncrementalVersion} \
                        versions:commit'
                    def matcher = readFile('pom.xml') =~ '<version>(.+)</version>'
                    def version = matcher[0][1]
                    env.IMAGE_NAME = "$version-$BUILD_NUMBER"
                }
            }
        }
        stage('build app') {
            steps {
                script {
                    echo "building the application..."
                    sh 'mvn clean package'
                }
            }
        }
        stage('build image') {
            steps {
                script {
                    echo "building the docker image..."
                    docker.withRegistry("", "dockerhubcredentiels") {
                        sh "docker build -t medali1996/javamaven:${IMAGE_NAME} ."
                        sh "docker push medali1996/javamaven:${IMAGE_NAME}"
                    }
                }
            }
        }
        stage('deploy') {
            steps {
                withKubeConfig([credentialsId: 'work',serverUrl: 'https://192.168.199.41:6443']) {
                    sh "sed -i 's#replace-image#medali1996/javamaven:${IMAGE_NAME}#g' deployment-java.yaml"
                    sh "kubectl apply -f deployment-java.yaml --namespace=default"
                }    
            }
        }
}
}
