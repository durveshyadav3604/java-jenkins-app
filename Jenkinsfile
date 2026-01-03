pipeline {
    agent any

    tools {
        maven 'maven-3.9.12'
        jdk 'jdk-21'
    }

    stages {

        stage("Check Tools") {
            steps {
                sh '''
                  echo "JAVA HOME = $JAVA_HOME"
                  java -version
                  mvn -version
                '''
            }
        }

        stage("Build Application") {
            steps {
                sh '''
                  mvn clean package
                '''
            }
        }
    }
}
