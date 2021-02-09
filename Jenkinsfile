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
				sh "mvn -f pom.xml clean install checkstyle:check"
			}
		}

		stage ('Analysis') {
			steps{
				recordIssues enabledForFailure: true, aggregatingResults: true, tool: checkStyle(pattern: 'target/checkstyle-result.xml', reportEncoding: 'UTF-8')
			}
		}
	}
}
