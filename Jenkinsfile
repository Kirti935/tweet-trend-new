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
        stage('build') {
            steps {
                echo "----------Build started----------"
                sh 'mvn clean deploy -Dmain.test.skip=true'
                echo "----------Build completed----------"
            }
        }
        stage('unit-test') {
            steps {
                echo "----------Running unit tests----------"
                sh 'mvn surefire-report:report'
                echo"----------Unit tests completed----------"
            }
        }

        stage('SonarQube analysis') {
            environment {
                scannerHome = tool 'kirti-sonar-scanner' //this is sonar scanner tool
            }
            steps {
                withSonarQubeEnv('kirti-sonarqube-server') { // this is sonarqube server 
                    sh "${scannerHome}/bin/sonar-scanner"
                }
            }
        }
        stage('SonarQube Quality Gate') {
            steps {
                scripts { 
                timeout(time: 1, unit: 'HOURS') //Just in case the quality gate takes a long time
                def qg = waitForQualityGate() // This will wait for the SonarQube quality gate result
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


