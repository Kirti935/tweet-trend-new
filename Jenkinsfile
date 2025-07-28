pipeline {
    agent {
        node { 
            label 'maven-slave'
        }
    }
environment {
        PATH = "/opt/apache-maven-3.9.2/bin:$PATH"

    }

    stages {
        stage('build') {
            steps {
                script {
                    // Clean the workspace
                    sh 'mvn clean'

                    // Build the project
                    sh 'mvn package'
                }
            }
        }
    }
}
