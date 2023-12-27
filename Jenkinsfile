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

stages {
        stage('Compile') {
			agent any
            steps {
			    script{
				echo "Compiling in ${params.ENV} environment"
				sh 'mvn compile'  
            }
          }
        }
		stage('UnitTest'){
			agent {label 'linux_slave'}
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

