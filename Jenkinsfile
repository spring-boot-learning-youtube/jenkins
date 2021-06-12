pipeline {
	agent none

	triggers {
		pollSCM 'H/10 * * * *' // triggered by polling source code on the 10th minute of the hour.
	}

	options {
		disableConcurrentBuilds() // No more than one job at a time is allowed to run.
		buildDiscarder(logRotator(numToKeepStr: '14')) // Only keep latest 14 jobs.
	}

	// Feel free to Google for "Jenkins pipeline syntax" to learn other available commands.

	stages {
		stage("test: baseline (jdk8)") {
			agent {
				docker {
					// Docker container to run this 'stage' inside
					image 'adoptopenjdk/openjdk8:latest'
					// Arguments fed to Docker container (i.e. maven caching preserved between jobs)
					args '-v $HOME/.m2:/tmp/jenkins-home/.m2' 
				}
			}
			options { timeout(time: 30, unit: 'MINUTES') } // if container freezes for 30 minutes, Jenkins will kill the job
			steps {
				sh 'echo "Run whatever shell steps you need to test your project"'
				sh './mvnw clean test'
			}
		}

	}

	post {
		changed {
			script {
				slackSend( // Send changes in CI job status to a slack channel
						color: (currentBuild.currentResult == 'SUCCESS') ? 'good' : 'danger',
						channel: '#your-slack-channel', // name of your Slack channel
						// Customize the slack message as desired.
						message: "${currentBuild.fullDisplayName} - `${currentBuild.currentResult}`\n${env.BUILD_URL}")

				emailext( // Send emails to relevant parties (whomever broke/fixed the pipeline)
						subject: "[${currentBuild.fullDisplayName}] ${currentBuild.currentResult}",
						mimeType: 'text/html',
						recipientProviders: [[$class: 'CulpritsRecipientProvider'], [$class: 'RequesterRecipientProvider']],
						// Customize the email message as desired
						body: "<a href=\"${env.BUILD_URL}\">${currentBuild.fullDisplayName} is reported as ${currentBuild.currentResult}</a>")
			}
		}
	}
}
