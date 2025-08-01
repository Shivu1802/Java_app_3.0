@Library('my-shared-library')

pipeline{
	agent any
	

	stages{

		stage("Git Checkout"){
				steps{
					script{
						if(params.action == 'create'){
							 	gitcheckout(
                    							branch: "main",
                    							url: "https://github.com/Shivu1802/Java_app_3.0.git"
                						)
							}
						else {
							echo 'pipeline not started...'
							}
						}
					}
				}	


		stage("parallel execution"){
			parallel{
				stage("Unit test"){
						steps{
							script{
								echo 'Running unit tests...'
								mvnTest()
								}
							}
						}
				stage("Integration test Maven"){
							steps{
								script{
									echo 'Running Integration tests...'
									mvnIntegrationTest()
								}
							}
						}
            }
        }

		stage("Static Code Analysis: Sonarqube"){
					steps{
						script{
							def SonarQubecredentialsId='sonarqube-api'
							staticCodeAnalysis(SonarQubecredentialsId)
							}
						}
					}

		stage("Quality Gate Status Check: SonarQube"){
							steps{
								script{
									def SonarQubecredentialsId='sonarqube-api'
									QualityGateStatus(SonarQubecredentialsId)
									}
								}
							}

		stage("Maven Build"){
				steps{
					script{
						mvnBuild()
						}
					}
				}

		stage("Manual Approval for Docker Build Image"){
							steps{
								script{
									def userInput = input( id: 'userInputID', 
												message: 'Do you want to proceed with Docker Image Build?',
												parameters: [ choice(name: 'Proceed', choices: ['Yes', 'No'], description: 'Select Yes to continue or No to stop the pipeline') 
												] 
											)
		if (userInput == 'No') {
   			error("Pipeline aborted by user.")
			}

										}
									}
								} 
								

		stage("Docker Image Build"){
				steps{
					script{
						dockerBuild("${params.ImageName}", "${params.ImageTag}", "${params.DockerHubUser}")
						}
					}
				}

		stage("Docker Image Scan: Trivy"){
						steps{
							script{
								dockerImageScan("${params.ImageName}", "${params.ImageTag}", "${params.DockerHubUser}")
							}
						}
					}
		
		  stage('Docker Image Push : DockerHub '){
            						steps{
               							script{
      									dockerImagePush("${params.ImageName}", "${params.ImageTag}", "${params.DockerHubUser}") 																	}
            							}
        						}

		stage("Docker Image Cleanup: DockerHub"){
							steps{
								script{
									dockerImageCleanup("${params.ImageName}", "${params.DockerHubUser}")
									}
								}
							}



		post {
    			always {
        			echo "Build Completed"
    				}
    			success {
        			echo "Build Success!"
    				}
    			failure {
        			echo "Build Failed!"
    				}

		}

    }

}
