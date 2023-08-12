#!/usr/bin/env groovy
def cleanUp()
{
    echo "Clean up Workspace"
    sh "rm -rf ${WORKSPACE}/.git"
    sh "rm -rf ${WORKSPACE}/*"
    // sh "docker system prune -af"
}

pipeline {
    agent {
        label 'docker-buildx'
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
                        sh """
                        wget https://github.com/docker-library/hello-world/blob/3fb6ebca4163bf5b9cc496ac3e8f11cb1e754aee/amd64/hello-world/hello
                        docker buildx build -t scratch-hello:${IMAGETAG} .
                        """
                        /*
                        sh "docker build -t acrmyshadowprepod.azurecr.io/myshadowbase:prod-${IMAGETAG} ."
                        sh "docker push acrmyshadowprepod.azurecr.io/myshadowbase:prod-${IMAGETAG}"
                        */
                    }
                    catch(Exception err) {
                        currentBuild.result = "FAILURE"
                        cleanUp()
                        throw err
                    }
                    
                }
            }

        }
        /*
        stage('Deploy myShadow to Production') {
            environment {
                IMAGE="acrmyshadowprepod.azurecr.io/myshadowbase:prod-${IMAGETAG}"
            }
            steps {
                script {
                    try {
                        dir("${WORKSPACE}/myshadow/") {
                            sh 'sed -i "s|BUILD_NUMBER|${BUILD_NUMBER}|g" Chart.yaml'
                            sh '''
                              az acr login --name acrmyshadowprepod
                              chmod 777 Chart.yaml
                              echo ${BUILD_NUMBER}
                              cat Chart.yaml
                              kubectl config use-context aks-myshadow-prod
                              echo "Image name - $IMAGE"
                              helm upgrade myshadow . --set image=${IMAGE}  --namespace prod-myshadow
                            '''
                        }
                    }
                    catch(Exception err) {
                        currentBuild.result = "FAILURE"
                        cleanup()
                        throw err
                    }
                }
            }
        }
        */
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
