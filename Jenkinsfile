def registry = 'https://kirti29.jfrog.io/'
def imageName = 'kirti29.jfrog.io/kirti-docker-local/ttrend'
def version = '2.1.2'

pipeline {
    agent {
        node { 
            label 'maven-slave'
        }
    }

    environment {
        PATH = "/opt/apache-maven-3.9.2/bin:$JAVA_HOME/bin:$PATH"
    }

    stages {
        stage('Build') {
            steps {
                echo "----------Build started----------"
                sh 'mvn clean deploy -Dmain.test.skip=true'
                echo "----------Build completed----------"
            }
        }

        stage('Unit Test') {
            steps {
                echo "----------Running unit tests----------"
                sh 'mvn test'
                sh 'mvn surefire-report:report'
                echo "----------Unit tests completed----------"
            }
        }
        
        stage('SonarQube Analysis') {
            environment {
                scannerHome = tool 'kirti-sonar-scanner' // Sonar Scanner tool name in Jenkins
            }
            steps {
                withSonarQubeEnv('kirti-sonarqube-server') { // SonarQube server name in Jenkins
                    sh "${scannerHome}/bin/sonar-scanner"
                }
            }
        }

        stage('SonarQube Quality Gate') {
            steps {
                script {
                    timeout(time: 1, unit: 'HOURS') {
                        def qg = waitForQualityGate()
                        if (qg.status != 'OK') {
                            error "Pipeline aborted due to quality gate failure: ${qg.status}"
                        } else {
                            echo "Quality gate passed: ${qg.status}"
                        }
                    }
                }
            }
        }

        stage("Jar Publish") {
            steps {
                script {
                    echo '<--------------- Jar Publish Started --------------->'
                    def server = Artifactory.newServer url: registry + "/artifactory", credentialsId: "jfrog-artifact-cred"

                    // Debug: List files in jarstaging directory
                    echo "Listing contents of jarstaging directory:"
                    sh 'ls -l jarstaging || echo "Directory not found or empty"'

                    // Define artifact properties
                    def properties = "buildid=${env.BUILD_ID},commitid=${GIT_COMMIT}";
                    def uploadSpec = """{
                        "files": [
                            {
                                "pattern": "jarstaging/**/*.jar",
                                "target": "kirti-maven-libs-release-local/",
                                "flat": "false",
                                "props" : "${properties}",
                                "exclusions": [ "*.sha1", "*.md5"]
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

        stage('Docker build') {
            steps {
                script {
                    echo "----------Docker build started----------"
                    app = docker.build(imageName + ":" + version)
                    echo "----------Docker build completed----------"
                }
            }
        }

        stage('Docker publish') {
            steps {
                script {
                    echo "----------Docker publish started----------"
                    docker.withRegistry(registry, 'jfrog-artifact-cred') {
                        app.push('latest')
                    }
                    echo "----------Docker publish completed----------"
                }
            }
        }
    }
}
