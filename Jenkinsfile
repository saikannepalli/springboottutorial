pipeline{
    agent any 
	 tools {
        maven 'Maven-3.6.1'
    } 
     stages{
//Build the Docker Image based on the Dockerfile
       stage(" Maven Clean Package"){
	  steps{
	    sh "mvn -Dmaven.test.failure.ignore=true install"
	  }
       }
        stage('Build Docker Image'){
	  steps{
	     sh "sudo docker build -t saikannepalli/springboottutorial:${BUILD_ID} ."   //when we run docker in this step, we're running it via a shell on the docker build-pod container
                        }
       }
 // Pushing the Docker Image to DockerHub   
     stage('Push Docker Image'){
	steps{
//Docker Hub Credentials
        withCredentials([string(credentialsId: 'DOKCER_HUB_PASSWORD', variable: 'sai-dockerpwd')]) {
          sh "sudo docker login -u saikannepalli -p ${sai-dockerpwd}"
          sh "sudo docker push saikannepalli/springboottutorial:${BUILD_ID}"
	              }
           }
       }    
     stage("Deploy To Kuberates Cluster"){
      steps{
//Created Service Account in GCP console and generated key is added it to the Jenkins Credentials.
        withCredentials([file(credentialsId: 'demo-key', variable: 'GOOGLE_APPLICATION_CREDENTIALS')]) {
//activate the service account
         sh "gcloud auth activate-service-account --key-file=${GOOGLE_APPLICATION_CREDENTIALS}"
//Configuring the project details to Jenkins and communicate with the gke cluster
         sh "gcloud config set project mssdevops-284216" //configuring name of the project
         sh "gcloud config set compute/zone us-central1-c"// confiuring the project compute-zone
         sh "gcloud config set compute/region us-central1"// confiuring the project compute-region
         sh "gcloud container clusters get-credentials cluster-1 --zone us-central1-c --project mssdevops-284216"// command to connect the GKE cluster through command line

         sh "sed -i -e 's,image_to_be_deployed,'saikannepalli/springboottutorial:${BUILD_ID}',g' springboot.yml"
         sh "kubectl apply -f springboot.yml"
    }
}

     }
     }
}
