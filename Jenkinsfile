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
                sh 'mvn clean deploy'
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
    }
}


