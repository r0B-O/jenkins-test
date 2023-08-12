#!/usr/bin/env groovy
pipeline {
    agent {
        kubernetes {
            defaultContainer 'docker-buildx'
            yaml '''
apiVersion: v1
kind: Pod
metadata:
  labels:
    run: buildx-pod
  name: buildx-pod
spec:
  containers:
  - image: docker:latest
    name: docker-buildx
    command:
      - "sh"
      - "-c"
      - |
        mkdir ~/.docker
        mkdir ~/.docker/cli-plugins && cd ~/.docker/cli-plugins 
        wget https://github.com/docker/buildx/releases/download/v0.5.1/buildx-v0.5.1.linux-amd64
        mv ./buildx-v0.5.1.linux-amd64 docker-buildx
        chmod +x ./docker-buildx
        docker buildx create --use --name k8s node-amd64 --driver kubernetes --driver-opt  image=moby/buildkit:master
        while true; do sleep 3600; done;
  restartPolicy: Never
  serviceAccount: buildx-sa
'''
        }
    }
    stages {
        stage("Checkout Source Code Branch") {
            environment {
                REPO_URL="https://github.com/r0B-O/jenkins-test.git"
            }
            steps {
                script {
                    try {
                        checkout([
                            $class: 'GitSCM',
                            branches: [[name: '*/main']],
                            doGenerateSubmoduleConfigurations: false,
                            extensions: [[
                                $class: 'CloneOption',
                                shallow: true,
                                depth: 1,
                                noTags: false
                            ]],
                            submoduleCfg: [],
                            userRemoteConfigs: [[
                                url: "$REPO_URL"
                            ]]
                        ])                       
                    }
                    catch(Exception err) {
                        currentBuild.result = "FAILURE"
                        cleanUp()
                        throw err
                    }
                }
            }
        }
        stage('Build myShadow Docker Image') {
            steps {
                script {
                    try {
                        TIMESTAMP = sh (
                           script: 'date +%Y%m%d%H%M%S',
                           returnStdout: true
                        ).trim()
                        IMAGETAG="${TIMESTAMP}"
                         withCredentials([usernameColonPassword(credentialsId: 'docker-creds', usernameVariable: 'docker-user', passwordVariable: 'docker-password')]) {
                            sh """
                            docker login -u ${docker-username} -p ${docker-password}
                            wget https://github.com/docker-library/hello-world/blob/3fb6ebca4163bf5b9cc496ac3e8f11cb1e754aee/amd64/hello-world/hello
                            docker buildx build -t r080/scratch-hello:${IMAGETAG} .
                            docker push r080/scratch-hello:${IMAGETAG}
                            """
                        }
                    }
                    catch(Exception err) {
                        currentBuild.result = "FAILURE"
                        cleanUp()
                        throw err
                    }
                    
                }
            }

        }
        stage('Cleanup Workspace') {
            steps {
                sh 'echo "Cleaning Up..."'
                cleanUp()
            }
        }
    }
    post {
        always {
            cleanUp()
        }
    }
}

def cleanUp()
{
    echo "Clean up Workspace"
    sh "rm -rf ${WORKSPACE}/.git"
    sh "rm -rf ${WORKSPACE}/*"
}
