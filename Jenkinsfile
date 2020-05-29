	// Obtaining an Artifactory server instance defined in Jenkins:

	def resolverServer = Artifactory.server 'Artifactory-server'
	def deployerServer = Artifactory.server 'Artifactory-server'
	def Server = Artifactory.server 'Artifactory-server'

			 //If artifactory is not defined in Jenkins, then create on:
			// def server = Artifactory.newServer url: 'Artifactory url', username: 'username', password: 'password'

	//Create Artifactory Maven Build instance
	def rtMaven = Artifactory.newMavenBuild()
	//def project_path = "spring-boot-samples/spring-boot-sample-velocity"

	def buildInfo

	pipeline {
	    agent any

		tools {
			//jdk "Java-1.8"
			maven "Maven-3.6.3"
		}

	    stages {
		stage('Clone sources'){
		    steps {
			git url: 'https://github.com/nandhakumarr94/jfrog2.git'
		    }
		}

		stage('SonarQube analysis') {
		     steps {
			//Prepare SonarQube scanner enviornment
			withSonarQubeEnv('Sonarqube_scanner') {
			   sh 'mvn org.sonarsource.scanner.maven:sonar-maven-plugin:3.3.0.603:sonar'
			}
		      }
		}

		stage('Quality Gate') {
			steps {
				timeout(time: 3, unit: 'Minutes') {
				//Parameter indicates wether to set pipeline to UNSTABLE if Quality Gate fails
				// true = set pipeline to UNSTABLE, false = don't
				// Requires SonarQube Scanner for Jenkins 2.7+
				waitForQualityGate abortPipeline: true
			       }
			 }
		}

		stage('Artifactory configuration') {

		   steps {
			script {
				rtMaven.tool = 'Maven-3.6.3' //Maven tool name specified in Jenkins configuration

				rtMaven.deployer releaseRepo: 'libs-release-local', snapshotRepo: 'libs-snapshot-local', server: deployerServer //Defining where the build artifacts should be deployed to

				rtMaven.resolver releaseRepo:'libs-release', snapshotRepo: 'libs-snapshot', server: resolverServer //Defining where Maven Build should download its dependencies from

				rtMaven.deployer.artifactDeploymentPatterns.addExclude("pom.xml") //Exclude artifacts from being deployed

				//rtMaven.deployer.deployArtifacts =false // Disable artifacts deployment during Maven run

				buildInfo = Artifactory.newBuildInfo() //Publishing build-Info to artifactory

				//buildInfo.retention maxBuilds: 10, maxDays: 7, deleteBuildArtifacts: true

				buildInfo.env.capture = true
				}
		    }
		}

		stage('Execute Maven') {

			steps {
			   script {
			// dir(project_path) {
				  // sh 'mvn clean verify'
			rtMaven.run pom: 'pom.xml', goals: 'clean install', buildInfo: buildInfo
				//}
			}

		}}

		stage('Publish build info') {
			steps {
			  script {

			Server.publishBuildInfo buildInfo
			}
			}
		}
	}
	}
