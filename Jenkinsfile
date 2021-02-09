pipeline{
	agent any

	tools {
		maven "3.6.3"
		jdk "jdk904"
	}

	stages{
		stage ('Checkout') {
			steps{
 				checkout([$class: 'GitSCM', branches: [[name: '*/master']], doGenerateSubmoduleConfigurations: false, extensions: [], submoduleCfg: [], userRemoteConfigs: [[credentialsId: '', url: 'https://github.com/edwardkolb/srccode-alt.git']]]) 
			}
		}
		
		stage('Build'){
			steps{
				sh "mvn -version"
				sh "mvn -f pom.xml clean install checkstyle:checkstyle"
			}
		}

		stage ('Analysis') {
			recordIssues enabledForFailure: true, aggregatingResults: true, tool: checkStyle(pattern: 'checkstyle-result.xml')
		}
	}
}
