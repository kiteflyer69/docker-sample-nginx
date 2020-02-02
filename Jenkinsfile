#!/usr/bin/env groovy

import hudson.FilePath;
import jenkins.model.Jenkins;

pipeline {
    agent {
        kubernetes {
            label 'jenkins-agent-nginx'
            yaml """
            apiVersion: v1
            kind: Pod
            metadata:
              labels:
                component: ci
            spec:
              containers:
                  - name: docker
                    image: docker:19.03.1
                    command:
                    - sleep
                    args:
                    - 99d
                    env:
                      - name: DOCKER_HOST
                        value: tcp://localhost:2375
                  - name: docker-daemon
                    image: docker:19.03.1-dind
                    securityContext:
                      privileged: true
                    env:
                      - name: DOCKER_TLS_CERTDIR
                        value: ""
                  - name: kubectl
                    image: lachlanevenson/k8s-kubectl:v1.15.5
                    command:
                      - cat
                    tty: true
            """
        }
    }

    stages {
        stage('Build image') {
            steps {
                container('docker') {
                    git 'https://github.com/kiteflyer69/docker-sample-nginx.git'
                    container('docker') {
                        sh 'docker version && DOCKER_BUILDKIT=1 docker build --progress plain -t kiteflyer69/nginx-example .'
                        sh 'docker images'
                        sh 'docker login -u="kiteflyer69" -p="admin1234"'
                        sh 'docker push kiteflyer69/nginx-example'
                    }
                }
            }
        }

        stage('Deploy') {
            steps {
                container('kubectl') {
                    sh "kubectl apply -f ./demo.yaml"
                }
            }
        }
    }
}
