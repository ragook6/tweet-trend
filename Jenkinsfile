pipeline {
    agent {
        node {
            label 'maven'
        }
    }
    environment {
        MAVEN_HOME = '/opt/apache-maven-3.9.4'
    }    
    stages {
        stage("Checkout") {
            steps {
                // Check out source code from Git
                checkout scm
            }
        }
        stage("Build") {
            steps {
                // Changing to the directory where pom.xml is located before running Maven
                dir('path/to/your/project') { // replace 'path/to/your/project' with the actual path relative to workspace
                    sh "${MAVEN_HOME}/bin/mvn clean deploy"
                }
            }
        }
    }
}