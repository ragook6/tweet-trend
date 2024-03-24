def registry = 'https://ragook6.jfrog.io'
def imageName = 'ragook6.jfrog.io/ragook6-docker-local/ttrend'
def version = '2.1.4'

pipeline {
    agent {
        node {
            label 'maven'
        }
    }
    environment {
        MAVEN_HOME = "/opt/apache-maven-3.9.4"
        JAVA_HOME_FOR_MAVEN = "/usr/lib/jvm/java-11-openjdk-amd64"
        JAVA_HOME_FOR_SONAR = "/usr/lib/jvm/java-17-openjdk-amd64"
    }
    stages {
        stage("Build") {
            steps {
                script {
                    env.PATH = "${env.MAVEN_HOME}/bin:${env.PATH}"
                    env.JAVA_HOME = "${env.JAVA_HOME_FOR_MAVEN}"
                }
                sh 'mvn clean deploy -Dmaven.test.skip=true'
            }
        }
        stage("Test") {
            steps {
                script {
                    env.PATH = "${env.MAVEN_HOME}/bin:${env.PATH}"
                    env.JAVA_HOME = "${env.JAVA_HOME_FOR_MAVEN}"
                }
                sh 'mvn surefire-report:report'
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
        stage("Quality Gate") {
            steps {
                script {
                    timeout(time: 1, unit: 'HOURS') {
                        def qg = waitForQualityGate()
                        if (qg.status != 'OK') {
                            error "Pipeline aborted due to quality gate failure: ${qg.status}"
                        }
                    }
                }
            }
        }
        stage("Jar Publish") {
            steps {
                script {
                    echo '<--------------- Jar Publish Started --------------->'
                    def server = Artifactory.newServer(url: registry + "/artifactory", credentialsId: "artifact-cred")
                    def properties = "buildid=${env.BUILD_ID},commitid=${GIT_COMMIT}"
                    def uploadSpec = """{
                        "files": [
                            {
                                "pattern": "jarstaging/(*)",
                                "target": "maven-libs-release-local/{1}",
                                "flat": "false",
                                "props" : "${properties}",
                                "exclusions": ["*.sha1", "*.md5"]
                            }
                        ]
                    }"""
                    def buildInfo = server.upload(uploadSpec)
                    buildInfo.env.collect()
                    server.publishBuildInfo(buildInfo)
                    echo '<--------------- Jar Publish Ended --------------->'
                }
            }
        }
        stage("Docker Build") {
            steps {
                script {
                    echo '<--------------- Docker Build Started --------------->'
                    app = docker.build(imageName + ":" + version)
                    echo '<--------------- Docker Build Ends --------------->'
                }
            }
        }
        stage("Docker Publish") {
            steps {
                script {
                    echo '<--------------- Docker Publish Started --------------->'
                    docker.withRegistry(registry, 'artifact-cred') {
                        app.push()
                    }
                    echo '<--------------- Docker Publish Ended --------------->'
                }
            }
        }
    }
}
