@Library('ceiba-jenkins-library') _
pipeline {
  //Donde se va a ejecutar el Pipeline
  agent {
    label 'Slave_Induccion'
  }

  //Opciones específicas de Pipeline dentro del Pipeline
  options {
    	buildDiscarder(logRotator(numToKeepStr: '3'))
 	disableConcurrentBuilds()
  }

  //Una sección que define las herramientas “preinstaladas” en Jenkins
  tools {
    jdk 'JDK8_Centos' //Verisión preinstalada en la Configuración del Master
  }
/*	Versiones disponibles
      JDK8_Mac
      JDK6_Centos
      JDK7_Centos
      JDK8_Centos
      JDK10_Centos
      JDK11_Centos
      JDK13_Centos
      JDK14_Centos
*/

  //Aquí comienzan los “items” del Pipeline
  stages{
     stage('Checkout') {
      steps{
        echo "------------>Checkout<------------"
		checkout scm
      }
    }
    
    stage('Compile & Unit Tests') {
      steps{
        echo "------------>Compile & Unit Tests<------------"
		sh 'chmod +x ./microservicio/gradlew'
		sh './microservicio/gradlew --b ./microservicio/build.gradle clean'
		sh './microservicio/gradlew --b ./microservicio/build.gradle test'
      }
    }

     stage('Static Code Analysis') {
      steps{
        echo '------------>Análisis de código estático<------------'
		withSonarQubeEnv('Sonar') {
				sh "${tool name: 'SonarScanner', type:'hudson.plugins.sonar.SonarRunnerInstallation'}/bin/sonar-scanner"
			}

      }
    }

    stage('Build') {
      steps {
        echo "------------>Build<------------"
		// Construir sin test
		sh './microservicio/gradlew --b ./microservicio/build.gradle build -x test'
      }
    }  
  }

  post {
    always {
      echo 'This will always run'
    }
    success {
      echo 'This will run only if successful'
	  junit '**/test-results/test/*.xml'  //RUTA DE LOS ARCHIVOS .XML
    }
    failure {
      echo 'This will run only if failed'
	  mail (to: 'oscar.olivo@ceiba.com.co',subject: "Failed Pipeline:${currentBuild.fullDisplayName}",body: "Something is wrong with ${env.BUILD_URL}")
    }
    unstable {
      echo 'This will run only if the run was marked as unstable'
    }
    changed {
      echo 'This will run only if the state of the Pipeline has changed'
      echo 'For example, if the Pipeline was previously failing but is now successful'
    }
  }
}