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
    gradle 'Gradle5.0_Centos'
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
//         checkout([
//             $class: 'GitSCM',
//             branches: [[name: '*/master']],
//             doGenerateSubmoduleConfigurations: false,
//             extensions: [],
//             gitTool: 'Default',
//             submoduleCfg: [],
//             userRemoteConfigs: [[
//             credentialsId: 'GitHub_verodriguezrSoft',
//             url:'https://github.com/verodriguezrSoft/backed-clinica-especializada-spring-boot'
//             ]]
//         ])

      }
    }

   stage('Compile & Unit Tests') {
      steps{

       echo "------------>Clean Tests<------------"

       sh 'chmod +x ./microservicio/gradlew'
       sh './microservicio/gradlew --b ./microservicio/build.gradle clean'


        echo "------------>compile & Unit Tests<------------"
        sh 'chmod +x ./microservicio/gradlew'
        sh './microservicio/gradlew --b ./microservicio/build.gradle test'
       }
    }

    stage('Static Code Analysis') {
      steps{

        withSonarQubeEnv('Sonar') {
			sh "${tool name: 'SonarScanner', type:'hudson.plugins.sonar.SonarRunnerInstallation'}/bin/sonar-scanner -Dproject.settings=sonar-project.properties"
	  	}
	  	echo '------------>Empiezo<------------'
		sonarqubeMasQualityGatesP(
		    sonarKey:'co.com.ceiba.adn:medicina.especializada.victor.rodriguez',
        	sonarName:'''"CeibaADN-MedicinaEspecializada(victor.rodriguez)"''',
        	sonarPathProperties:'./sonar-project.properties')
		echo '------------>Termino<------------'
      }
    }

    stage('Build') {
      steps {
		echo "------------>Build<------------"
		//Construir sin tarea test que se ejecutó previamente
		sh 'chmod +x ./microservicio/gradlew'
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
      junit 'microservicio/infraestructura/build/test-results/test/*.xml'
      junit 'microservicio/dominio/build/test-results/test/*.xml'
    }
    failure {
      echo 'This will run only if failed'
      mail (to: 'victor.rodriguez@ceiba.com.co',subject: "Failed Pipeline:${currentBuild.fullDisplayName}",body: "Something is wrong with ${env.BUILD_URL}")
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