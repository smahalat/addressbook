pipeline {
    agent none
    tools{
        maven 'myMaven'
        jdk 'MyJava'
    }

    parameters{
         string(name: 'ENV', defaultValue: 'DEV', description: 'env to compile')
         booleanParam(name: 'executeTest', defaultValue: true, description: 'decide to run tc')
         choice(name: 'APPVERSION', choices: ['1.1', '1.2', '1.3'], description: 'Pick app version')
    }

    environment{
        BUILD_SERVER='ec2-user@172.31.13.160'
        IMAGE_NAME='sagarmahalatkar/java-mvn-privaterepos'
        DEPLOY_SERVER='ec2-user@172.31.15.43'
    }

    stages {
        stage('Compile') {
            agent any
            steps {
                script{   
               // sshagent(['build-server']) {
                         
                echo "Compiling in ${params.ENV} environment"
                sh 'mvn compile'
             //   sh "scp -o StrictHostKeyChecking=no server-config.sh ${BUILD_SERVER}:/home/ec2-user"
             //   sh "ssh -o StrictHostKeyChecking=no ${BUILD_SERVER} 'bash ~/server-config.sh'"   
                
           // }
            }
            } 
        }
        stage("UnitTest"){
            agent any
            when{
                expression{
                    params.executeTest == true
                }
            }
            steps{
                script{
                echo "Run the Unit test cases"
                sh 'mvn test'
            }
            }
            post{
                always{
                    junit 'target/surefire-reports/*.xml'
                }
            }
        }
        stage("Dockerise"){
            agent any
            steps{
                script{
               sshagent(['build-server']) {
               withCredentials([usernamePassword(credentialsId: 'docker-hub', passwordVariable: 'PASSWORD', usernameVariable: 'USERNAME')]) {          
                echo "Containerising in ${params.ENV} environment"
                //sh 'mvn compile'
                sh "scp -o StrictHostKeyChecking=no server-config.sh ${BUILD_SERVER}:/home/ec2-user"
                sh "ssh -o StrictHostKeyChecking=no ${BUILD_SERVER} 'bash ~/server-config.sh ${IMAGE_NAME} ${BUILD_NUMBER}'"
                sh "ssh ${BUILD_SERVER} sudo docker login -u ${USERNAME} -p ${PASSWORD}"
                sh "ssh ${BUILD_SERVER} sudo docker push ${IMAGE_NAME}:${BUILD_NUMBER}"

            }
                }
                }
            }
        }
        stage("Deploy"){
            agent any
            input{
                message "Select the version to deploy"
                ok "Version selected"
                parameters{
                    choice(name: 'NEWAPP',choices:['EKS','ONPrem','Ec2'])
                }
            }
            steps{
                script{
                 sshagent(['build-server']) {
                withCredentials([usernamePassword(credentialsId: 'docker-hub', passwordVariable: 'PASSWORD', usernameVariable: 'USERNAME')]) {          
                echo "Deploying in ${params.ENV} environment"
                //sh 'mvn compile'
                
                sh "ssh -o StrictHostKeyChecking=no ec2-user@${DEPLOY_SERVER} sudo yum install docker -y"
                sh "ssh ec2-user@${DEPLOY_SERVER} sudo systemctl start docker"
                sh "ssh ec2-user@${DEPLOY_SERVER} sudo docker login -u ${USERNAME} -p ${PASSWORD}"
                sh "ssh ec2-user@${DEPLOY_SERVER} sudo docker run -itd -p 8001:8080 ${IMAGE_NAME}:${BUILD_NUMBER}"
                }
            }

                }
            }
        }
