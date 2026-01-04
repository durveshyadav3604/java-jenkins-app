pipeline {
 agent any


parameters {
   string(name: 'DEPLOY_ENV', defaultValue: 'developmeent', description: 'Select the target environment')
}

stages{

 stage("checkout"){
     steps {
           sh """
           echo "Checkout done - $PWD"
           echo "DEPOLYMENT ENV SELECTED - $DEPLOY_ENV"
           ls -l
           """
      }
   }

 
     stage("Building the application"){
     steps {
         
        sh 'echo "========Building Java Application============"'
        sh '/opt/apache-maven-3.9.12/bin/mvn -v'
        sh '/opt/apache-maven-3.9.12/bin/mvn clean package -B'
        sh 'echo "======Building Java Application completed====="'
    
      }
   }
           
} // end of stages

} // end of pipeline
