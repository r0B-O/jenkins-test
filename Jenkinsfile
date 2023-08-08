#!/usr/bin/env groovy
properties([
    parameters([
        string(defaultValue: "prod", description: 'Which Git Branch to clone?', name: 'GIT_BRANCH')
    ])
])

def cleanUp()
{
    echo "Clean up Workspace"
    sh "rm -rf ${WORKSPACE}/.git"
    sh "rm -rf ${WORKSPACE}/*"
    sh "docker system prune -af"
}

pipeline {
    agent any  
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
                        sh 'az acr login --name acrmyshadowprepod'                                
                        GIT_COMMIT_ID = sh (
                           script: 'git log -1 --pretty=%H',
                           returnStdout: true
                        ).trim()
                        TIMESTAMP = sh (
                           script: 'date +%Y%m%d%H%M%S',
                           returnStdout: true
                        ).trim()
                        echo "Git commit id: ${GIT_COMMIT_ID}"
                        IMAGETAG="${GIT_COMMIT_ID}-${TIMESTAMP}"
                        sh "docker build -t nodejs-hello:${IMAGETAG}"
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
