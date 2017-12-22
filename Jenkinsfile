node{

     // Poll every minute
     properties([pipelineTriggers([pollSCM('* * * * *')])])

     String GIT_SHORT_COMMIT
     String GIT_LONG_COMMIT
  

        stage ('GITHUB Checkout EI FrontEnd Artifact SC') {
	    
		    dir ('sourcecode') {
	                        
									   
                            git poll: true, branch: "master", url: 'https://github.com/emichaf/eiffel-intelligence-frontend.git'
                           
							GIT_SHORT_COMMIT = sh(returnStdout: true, script: "git log -n 1 --pretty=format:'%h'").trim()

                            GIT_LONG_COMMIT =  sh(returnStdout: true, script: "git log --format='%H' -n 1").trim()
            }
              
			  
        }

	        
		     
		stage ('GERRIT Update Build Info and Push change') {

			dir ('wrapper') {
			
			
			// OBS do NOT use git poll: false... it will not work, the build will be triggered circular after the 
			// build info file been updated and pushed
			
			checkout scm: [$class: 'GitSCM', 
				  userRemoteConfigs: [[url: 'https://github.com/emichaf/eiffel-intelligence-frontend-artifact-wrapper.git']],
				  branches: [[name: '*/master']]], changelog: false, poll: false
	
	
                   withCredentials([[$class: 'UsernamePasswordMultiBinding',
                                    credentialsId: 'fbb60332-6a43-489a-87f7-4cea380ad6ca',
                                    usernameVariable: 'GITHUB_USER',
                                    passwordVariable: 'GITHUB_PASSWORD']]) {

							
							
							
							// Update build_info.yaml file with github hash
							String file_name = 'build_info.yaml'
							def exists = fileExists "$file_name"
								if (exists) {
									sh "rm $file_name"
								} 
								
							def yaml_content = ['commit': "$GIT_LONG_COMMIT"]                                                                          
                            writeYaml file: "$file_name", data: yaml_content
							

                            sh('git config user.email ${GITHUB_USER}')
                            sh('git config user.name ${GITHUB_USER}')


                            sh('git add .')
                            sh('git commit -m "build info updated"')

                            sh("git push http://${GITHUB_USER}:${GITHUB_PASSWORD}@github.com/emichaf/eiffel-intelligence-frontend-artifact-wrapper.git HEAD:master")

                   }
				   
            }

        }
		
		
		
		stage ('Trigger EI FrontEnd component jobs') {
		
		    build job: 'eiffel-intelligence-frontend-artifact-wrapper/master', parameters: [[$class: 'StringParameterValue', name: 'param1', value: 'test_param']]
				
        }
		


     
	   
 }

