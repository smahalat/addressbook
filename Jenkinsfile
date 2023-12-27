pipeline {
    agent none
	tools{
        maven 'myMaven'
        jdk 'MyJava'
    }
	parameters{
		string(name: 'ENV', defaultValue: 'DEV', description: 'Environment to Compile')
		booleanParam(name: 'ExecuteTest', defaultValue: true, description: 'Decide to run TC or not?')
		choice(name: 'APPVERSION', choices: ['1.1', '1.2', '1.3'], description: 'Pick APP Version')
	}
 environment{
	  BUILD_SERVER='ec2-user@172.31.38.109'
 }
stages {
        stage('Compile') {
		 steps {
			script{
			       sshagent(['build-server']) {
           
				echo "Compiling in ${params.ENV} environment"
				// sh 'mvn compile'
				sh "scp -o StrictHostKeyChecking=no server-config.sh ${BUILD_SERVER}:/home/ec2-user"
				sh "ssh -o StrictHostKeyChecking=no ${BUILD_SERVER} 'bash ~/server-config.sh'"  
            }
          }
			}
        }
		stage('UnitTest'){
			agent {label 'linux_Slave'}
			when{
				expression{
					params.ExecuteTest == true
				}
			}
		    steps{
			   script{
			      echo"Run the UnitTest cases"
			      sh 'mvn test'
			   }
			}
			post{
				always { 
                                        junit 'target/surefire-reports/*.xml'
        				}
			     }
		}
		stage('Package'){
			agent any
		    steps{
			  script{
				echo"Packaging the App version ${params.APPVERSION}"
				  sh 'mvn package'
			  }
			
			}
		}
	stage('Deploy'){
		agent any
		input{
		      message "select the version to deploy"
			  ok "Version selected"
			  parameters{
			    choice(name:'NEWAPP', choices:['EKS', 'OnPrem', 'EC2'])
			  }
		}
        steps{
		    script{
			  echo"Deploy the code"  
			  echo"Deploy the app to ${NEWAPP}"
			}
		}
	}	
    }
}
