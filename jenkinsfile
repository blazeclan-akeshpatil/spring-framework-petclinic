pipeline {
  agent {
    node {
      label 'maven-slave'
    }
  }
  stages {
    stage('Maven Build') {
      steps {
        checkout([
		$class: 'GitSCM',
		branches: [[name: '*/main']],
		extensions: [],
		userRemoteConfigs: [[url: 'https://github.com/blazeclan-akeshpatil/spring-framework-petclinic.git']]
		])
        sh 'mvn -U clean install'
      }
    }
    stage('Quality Check of codes with SonarQube') {
      steps {
      	withSonarQubeEnv('sonar') {
	  sh "mvn sonar:sonar"
	}
      }
    }
    stage("Quality gate") {
	 steps {
	   waitForQualityGate abortPipeline: true
 	}
    }
    stage('Push the Artifact into Nexus') {
      steps {
      	nexusArtifactUploader artifacts: [
		[
			artifactId: 'spring-framework-petclinic',
			classifier: '',
			file: 'target/petclinic.war',
			type: 'war'
		]
	],
	credentialsId: 'nexusauth',
	groupId: 'org.springframework.samples',
	nexusUrl: 'nexus:8081',
	nexusVersion: 'nexus3',
	protocol: 'http',
	repository: 'petclinic',
	version: '${BUILD_NUMBER}'
      }
    }
    stage('Pull the Artifact from Nexus and Deploy on Production') {
      agent any
      steps {
      	sh 'curl -X GET http://admin:admin@nexus:8081/nexus/service/local/repositories/petclinic/content/org/springframework/samples/${BUILD_NUMBER}/petclinic-${BUILD_NUMBER}.war -o petclinic.war'
	deploy adapters: [tomcat9(credentialsId: 'tomcat-cred', path: '', url: 'http://tomcat:9090')], contextPath: 'DevOpsDemo', war: '**/*.war'
      }
    }
  }
}
