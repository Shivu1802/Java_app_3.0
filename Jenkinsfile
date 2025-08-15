@Library("my-shared-library") _

pipeline {

	agent any

	  parameters{

        choice(name: 'action', choices: 'create\ndelete', description: 'Choose create/Destroy')
        string(name: 'ImageName', description: "name of the docker build", defaultValue: 'javapp')
        string(name: 'ImageTag', description: "tag of the docker build", defaultValue: 'v1')
        string(name: 'DockerHubUser', description: "name of the Application", defaultValue: 'shivamsaini1802')
    }

	stages {

		stage("Git Checkout") {
			steps {
				script {
		if (params.action == 'create') {
			git branch: "main", url: "https://github.com/Shivu1802/Java_app_3.0.git"
		}
		else {
			echo "Skipping Git Checkout as action is set to delete."
		}
				}
			}
		}

		
		stage("Unit Test:maven") {
					when {
						expression { params.action == 'create' }
					}
					steps {
						script {
							mvnTest()
						}
					}
				}

		stage("Integration Test:maven") {
					when {
						expression { params.action == 'create' }
					}
					steps {
						script {
							mvnIntegrationTest()
						}
					}
				}
			
		
		

		stage("Static Code Analysis: Sonarqube") {
			steps {
				script {
					def SonarQubecredentialsId = 'sonarqube-api'
					staticCodeAnalysis(SonarQubecredentialsId)
				}
			}
		}

		stage("Quality Gate Status Check: SonarQube") {
			steps {
				script {
					def SonarQubecredentialsId = 'sonarqube-api'
					QualityGateStatus(SonarQubecredentialsId)
				}
			}
		}

		stage("Maven Build") {
			when {
				expression { params.action == 'create' }
			}
			steps {
				script {
					mvnBuild()
				}
			}
		}

		stage("Manual Approval for Docker Build Image") {
			steps {
				script {
					def userInput = input(
						id: 'userInputID',
						message: 'Do you want to proceed with Docker Image Build?',
						parameters: [
							choice(
								name: 'Proceed',
								choices: ['Yes', 'No'],
								description: 'Select Yes to continue or No to stop the pipeline'
							)
						]
					)
					if (userInput == 'No') {
						error("Pipeline aborted by user.")
					}
				}
			}
		}

		stage("Docker Image Build") {
			when {
				expression { params.action == 'create' }
			}
			steps {
				script {
					dockerBuild(params.ImageName, params.ImageTag, params.DockerHubUser)
				}
			}
		}

		stage("Docker Image Scan - Trivy") {
			when {
				expression { params.action == 'create' }
			}
			steps {
				script {
					dockerImageScan(params.ImageName, params.ImageTag, params.DockerHubUser)
				}
			}
		}

		stage('Docker Image Push : DockerHub ') {
			when {
				expression { params.action == 'create' }
			}
			steps {
				script {
					dockerImagePush("${params.ImageName}", "${params.ImageTag}", "${params.DockerHubUser}")
				}
			}
		}

		stage("Docker Image Cleanup : DockerHub ") {
			when {
				expression { params.action == 'create' }
			}
			steps {
				script {
					dockerImageCleanup("${params.ImageName}", "${params.ImageTag}", "${params.DockerHubUser}")
				}
			}
		}

		// post {
		// 	always {
		// 		echo "Build Completed"
		// 	}
		// 	success {
		// 		echo "Pipeline completed successfully."
		// 	}
		// 	failure {
		// 		echo "Build Failed! Please refer to the troubleshooting guide at https://github.com/Shivu1802/Java_app_3.0/wiki/Troubleshooting or check the build logs for details."
		// 	}
		// }


	}
}
