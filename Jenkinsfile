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
    }
}
