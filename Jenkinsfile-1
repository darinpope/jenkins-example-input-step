pipeline {
    agent any
    environment {
        git_credential = "github"
        aws_credential = "jenkins-s3-push"
        repo_url = "https://github.com/arunbeniwal008/pipeline-aws-plugin.git"
        base_imagename = "base-image"
        api_imagename = "my-api"
        auth_imagename = "my-auth"
        bucket = "aruns3jenkins"
        region = "ap-south-1"
        webHook_url = "myWebHookURL"
        api_res_url = "https://${bucket}.s3.${region}.amazonaws.com/${TAG_NAME}/${api_imagename}-${TAG_NAME}.tar.gz"
        auth_res_url = "https://${bucket}.s3.${region}.amazonaws.com/${TAG_NAME}/${auth_imagename}-${TAG_NAME}.tar.gz"
        notify_text = "image upload to s3 <br>${api_imagename}: <${api_res_url}><br> ${auth_imagename}: <${auth_res_url}><br>tag by ${TAG_NAME}"
        
    }
    
    tools {
        // Install the Maven version and add it to the path.
        maven "maven-3.3.9"
    }
stages {
        stage('checkout') {
            steps {
                script {
                    git branch:"${BRANCH_NAME}",
                        credentialsId: "${git_credential}",
                        url: "http://${repo_url}"
                    commitId = sh (script: 'git rev-parse --short HEAD ${GIT_COMMIT}', returnStdout: true).trim()
}
            }
        }
        stage("Maven build"){
            steps {
                // Run Maven on a Unix agent.
                sh "mvn clean install -DskipTests=true -Dmaven.test.failure.ignore=true."
            }
        }
        stage("Image build"){
            steps {
                script {
                    sh "docker images"
                    dir("${base_imagename}"){
                        docker.build "${base_imagename}"
                    }
                    dir("${api_imagename}"){
                        docker.build "${api_imagename}"
                    }
                    dir("${auth_imagename}"){
                        docker.build "${auth_imagename}"
                    }
                    sh "docker images"
                }
            }
        }
        stage("Zip"){
            steps{
                sh "mkdir ${TAG_NAME}"
                dir("${TAG_NAME}"){
                    sh "docker save ${api_imagename}:latest > ${api_imagename}-${TAG_NAME}.tar.gz"
                    sh "docker save ${auth_imagename}:latest > ${auth_imagename}-${TAG_NAME}.tar.gz"
                }
            }
        }
        stage("Upload"){
            steps{
                withAWS(region:"${region}", credentials:"${aws_credential}){
                    s3Upload(file:"${TAG_NAME}", bucket:"${bucket}", path:"${TAG_NAME}/")
                }    
            }
            post {
                success{
                    office365ConnectorSend message: "${notify_text}<br>commit id: ${commitId}", status:"Success Upload", webhookUrl:"${webHook_url}"
     sh "ls"
                }
                failure{
                    office365ConnectorSend message: "Fail build,<br> see (<${env.BUILD_URL}>)", status:"Fail Upload", webhookUrl:"${webHook_url}"
                }
            }
        }
        stage('Push Tag') {
            steps {
                script {
                    datetime = new Date().format("yyyy-MM-dd HH:mm:ss");
                    withCredentials([[$class: 'UsernamePasswordMultiBinding', credentialsId: "${git_credential}", usernameVariable: 'GIT_USERNAME', passwordVariable: 'GIT_PASSWORD']]) {
                        sh("git tag -a ${TAG_NAME} -m '${datetime}'")
                        sh("git push http://${env.GIT_USERNAME}:${env.GIT_PASSWORD}@${repo_url} --tags")
                    }
                }
            }
        }
    }
    post {
        always {
            cleanWs()
            dir("${env.WORKSPACE}@tmp") {
              deleteDir()
            }
            dir("${env.WORKSPACE}@script") {
              deleteDir()
            }
            dir("${env.WORKSPACE}@script@tmp") {
              deleteDir()
            }
            sh "docker rmi ${base_imagename}"
            sh "docker rmi ${api_imagename}"
            sh "docker rmi ${auth_imagename}"
        }
    }
}
