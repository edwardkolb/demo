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
            steps {
                sh '${M2_HOME}/bin/mvn --batch-mode -V -U -e checkstyle:check'
            }
        
    
			post {
				always {
					junit testResults: '**/target/surefire-reports/TEST-*.xml'
					recordIssues enabledForFailure: true, tool: checkStyle()
				}
			}
		}
	}
}
