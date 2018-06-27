node{

     // Poll every minute
     properties([pipelineTriggers([pollSCM('* * * * *')])])

     String GIT_LONG_COMMIT


     String SOURCE_CODE_REPO_URI = "https://github.com/ericsson/eiffel-intelligence-frontend.git"
     String SOURCE_CODE_BRANCH = "master"

     String WRAPPER_REPO = "https://github.com/emichaf/eiffel-intelligence-frontend-artifact-wrapper.git"
     String WRAPPER_REPO_PATH = "github.com/emichaf/eiffel-intelligence-frontend-artifact-wrapper.git"
     String build_info_file = 'build_info.yml'

     String WRAPPER_PIPELINE = "eiffel-intelligence-frontend-artifact-wrapper"
     String WRAPPER_BRANCH = "master"




     String SOURCE_NAME = "Jenkins"

        stage ('GITHUB Checkout EI BackEnd Artifact SC') {

		    dir ('sourcecode') {

                            git poll: true, branch: "$SOURCE_CODE_BRANCH", url: "$SOURCE_CODE_REPO_URI"

                            GIT_LONG_COMMIT =  sh(returnStdout: true, script: "git log --format='%H' -n 1").trim()

            }


        }



		stage ('GERRIT - Update Wrapper Build Info and Push Change') {

			dir ('wrapper') {


			// OBS do NOT use git poll: false... it will not work, the build will be triggered circular after the
			// build info file been updated and pushed

			checkout scm: [$class: 'GitSCM',
				  userRemoteConfigs: [[url: "$WRAPPER_REPO"]],
				  branches: [[name: '*/master']]], changelog: false, poll: false


                   withCredentials([[$class: 'UsernamePasswordMultiBinding',
                                    credentialsId: 'GITHUB_CREDENTIALS',
                                    usernameVariable: 'GITHUB_USER',
                                    passwordVariable: 'GITHUB_PASSWORD']]) {




							// Update build_info.yml file with github hash
							def exists = fileExists "$build_info_file"
								if (exists) {
									sh "rm $build_info_file"
								}

							def yml_content = ['commit': "$GIT_LONG_COMMIT"]
                            writeYaml file: "$build_info_file", data: yml_content


                            sh('git config user.email ${GITHUB_USER}')
                            sh('git config user.name ${GITHUB_USER}')


                            sh('git add .')
                            sh('git commit -m "build info file updated"')

                            sh("git push http://${GITHUB_USER}:${GITHUB_PASSWORD}@${WRAPPER_REPO_PATH} HEAD:${WRAPPER_BRANCH}")

                   }

            }

        }




	stage ('Trigger Wrapper Builds') {

		    build job: "${WRAPPER_PIPELINE}/${WRAPPER_BRANCH}", parameters: [[$class: 'StringParameterValue', name: 'param1', value: 'test_param']]

        }


         // Clean up workspace
         step([$class: 'WsCleanup'])


 }


