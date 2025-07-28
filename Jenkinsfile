pipeline {
    agent {
        node { 
            label 'maven-slave'
        }
    }
    environment {
        //JAVA_HOME = "/usr/lib/jvm/java-8-openjdk-amd64"
        PATH = "/opt/apache-maven-3.9.2/bin:$JAVA_HOME/bin:$PATH"
    }
    stages {
        stage('build') {
            steps {
                sh 'mvn clean deploy'
            }
        }
    }
}
