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
        def mvnHome = tool 'mvn-default'

        sh "${mvnHome}/bin/mvn -batch-mode -V -U -e checkstyle:checkstyle pmd:pmd pmd:cpd findbugs:findbugs spotbugs:spotbugs"

        def checkstyle = scanForIssues tool: [$class: 'CheckStyle'], pattern: '**/target/checkstyle-result.xml'
        publishIssues issues:[checkstyle], unstableTotalAll:1

        def pmd = scanForIssues tool: [$class: 'Pmd'], pattern: '**/target/pmd.xml'
        publishIssues issues:[pmd], unstableTotalAll:1

        def cpd = scanForIssues tool: [$class: 'Cpd'], pattern: '**/target/cpd.xml'
        publishIssues issues:[cpd]

        def findbugs = scanForIssues tool: [$class: 'FindBugs'], pattern: '**/target/findbugsXml.xml'
        publishIssues issues:[findbugs], unstableTotalAll:1

        def spotbugs = scanForIssues tool: [$class: 'SpotBugs'], pattern: '**/target/spotbugsXml.xml'
        publishIssues issues:[spotbugs], unstableTotalAll:1

        def maven = scanForIssues tool: [$class: 'MavenConsole']
        publishIssues issues:[maven]
		}
	}
}
