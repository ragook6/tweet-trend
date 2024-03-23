pipeline {
    agent {
        node {
            label 'maven'
        }
    }
    environment {
        MAVEN_HOME = "/opt/apache-maven-3.9.4"
        JAVA_HOME_FOR_MAVEN = "/usr/lib/jvm/java-11-openjdk-amd64" // Adjust this path to your Java 11 installation
        JAVA_HOME_FOR_SONAR = "/usr/lib/jvm/java-17-openjdk-amd64" // Adjust this path to a Java version compatible with your SonarQube
    }    
    stages {
        stage("Build") {
            steps {
                script {
                    env.PATH = "${env.MAVEN_HOME}/bin:${env.PATH}"
                    env.JAVA_HOME = "${env.JAVA_HOME_FOR_MAVEN}"
                }
                sh 'mvn clean deploy'
            }
        }
        stage('SonarQube analysis') {
            environment {
                scannerHome = tool 'ragook6-sonar-scanner'
            }
            steps {
                script {
                    env.JAVA_HOME = "${env.JAVA_HOME_FOR_SONAR}"
                }
                withSonarQubeEnv('ragook6-sonarqube-server') {
                    sh "${scannerHome}/bin/sonar-scanner"
                }
            }
        }
    }
}

