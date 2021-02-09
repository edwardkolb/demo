pipeline{
	agent any

	tools {
		maven "3.6.3"
		jdk "jdk904"
	}

	stages{
		stage ('Checkout') {
			steps{
 				checkout([$class: 'GitSCM', branches: [[name: '*/master']], doGenerateSubmoduleConfigurations: false, extensions: [], submoduleCfg: [], userRemoteConfigs: [[credentialsId: '', url: 'https://github.com/edwardkolb/demo-1.git']]]) 
			}
		}
		
		stage('Build'){
			steps{
				sh "mvn -version"
				sh "mvn -f pom.xml clean install"
			}
		}
	}

	post{
		always{
			cleanWs()

		}
	}
}
