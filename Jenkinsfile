pipeline {
	agent any
	options {
		timestamps()
	}
	stages {
		stage('Initialize') {
			steps {
				script {
					// env.YAML_LOCATION = "${env.JENKINS_WORKSPACE_LOCATION}/ruda_args_kubernetes/TestCODAI.yml"
					env.REPO_LOWER_NAME = sh(script: 'echo "$GIT_URL" | sed -En \'s/.*.com\\/(.+?).git/\\1/p\' | tr \'[:upper:]\' \'[:lower:]\'', returnStdout: true).trim()
					env.IMAGE_TAG = "${REGISTRY_PATH}/${env.REPO_LOWER_NAME}"
					env.GIT_NAME=sh(script: 'git --no-pager show -s --format="%an" "$GIT_COMMIT"', returnStdout: true).trim()
					env.GIT_EMAIL=sh(script: 'git --no-pager show -s --format="%ae" "$GIT_COMMIT"', returnStdout: true).trim()

					def gitCommitMessage = sh(script: 'git log -1 --pretty=%B', returnStdout: true).trim()
					currentBuild.displayName = "${env.REPO_LOWER_NAME}:${env.GIT_BRANCH} - ${env.GIT_COMMIT}: ${gitCommitMessage}"
				}
			}
		}
		stage('Git') {
			steps {
				git credentialsId: '640379b5-1f5f-4560-9d3a-6d76c06f0cf3', url: "${GIT_URL}"
			}
		}
		stage('Build') {
			steps {
				withDockerRegistry(credentialsId: 'docker-registry-credential', url: 'https://45.121.161.72:30001') {
          sh "docker build -t ${IMAGE_TAG} ."
					sh "docker tag ${IMAGE_TAG} ${IMAGE_TAG}:${GIT_COMMIT}"
          sh "docker push ${IMAGE_TAG} && docker push ${IMAGE_TAG}:${GIT_COMMIT}"
        }
				script {
					sh "echo GIT_COMMIT ${GIT_COMMIT}"
					sh "echo GIT_BRANCH ${GIT_BRANCH}"
					sh "echo GIT_URL ${GIT_URL}"
					sh "echo GIT_NAME ${GIT_NAME}"
					sh "echo GIT_EMAIL ${GIT_EMAIL}"
					// def dirname = sh(script: "dirname ${YAML_LOCATION}", returnStdout: true).trim()
					// sh "cd ${dirname} && git reset --hard HEAD && git pull origin main"
					// sh "sed 's|image:.*|image: '${REGISTRY_PATH}'/'${REPO_LOWER_NAME}':'${GIT_COMMIT}'|' ${YAML_LOCATION} > /tmp/args_'${JOB_NAME}'_'${GIT_COMMIT}'"
				}
			}
		}
		// stage("Trigger Downstream Project") {
		// 	steps {
		// 		script {
		// 			build job: "${DOWNSTREAM_PRJ}", parameters: [string(name: "COMMIT_MESSAGE", value: "${JOB_NAME}-${GIT_COMMIT}"), string(name: "FILE_CHANGE", value: "${YAML_LOCATION}"), string(name: "FILE_CONTENT", value: "/tmp/args_${JOB_NAME}_${GIT_COMMIT}")]
		// 		}
		// 	}
		// }
	}
}