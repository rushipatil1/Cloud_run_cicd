node('Prod'){
    startedby = env.BUILD_USER
  
    
    //get project id based on envirement 
    def project_id = ""
    def service_name = ""
    if(environment == "DEV"){
        project_id="mydev"
        service_name = "myaudio-test"
    }
    if(environment == "PROD"){
        project_id="mygcp"
        service_name = "myaudio-ai-clrun-prod"
    }
    
    def image_name_latest = "gcr.io/${project_id}/${service_name}:latest"
    def image_name = "gcr.io/${project_id}/${service_name}:${build_name}"
    def keyFilePath = "/home/Rushikesh.Patil/Jenkins/keys/${project_id}/.key.json"
    

      stage('clean workspace'){
         cleanWs()
         if ("${env.environment}" == "PROD" || "${env.environment}".toString().contains("prod")) {
            timeout(45) {
                input message: "Approve for ${env.environment}", submitter: 'rushikesh.patil, jyotsna.desale'
            }
        }
      }
      
      stage('Clone code Ripository'){
          git branch: "${branch}", credentialsId: '9a0cf526-12ca-400f-133b-86011049d911', url: 'http://10.5.4.28:8490/scm/ai/myaudio-ai.git'
      }
      stage('Clone code Ripository'){

        withCredentials([usernamePassword(credentialsId: '9a0cf526-12ca-400f-133b-86011049d911', passwordVariable: 'passwd', usernameVariable: 'username')]) {
            sh """
                git clone -b master http://${username}:${passwd}@10.5.4.28:8490/scm/an/cloudrun.git
                pwd
                ls -lahR
            """
        }
     }
      stage('login to gcloud'){
            sh """
                gcloud auth activate-service-account --key-file ${keyFilePath}
                gcloud config set project ${project_id}
                gcloud auth configure-docker
            """
      }
      stage('create and deploy docker image to CR'){
          sh """
            docker build -t ${image_name} .
            docker push ${image_name}
            docker tag ${image_name} ${image_name_latest}
            docker push ${image_name_latest}
          """
    }
    stage('deploy app to GCP'){
        sh """
         ansible-playbook -e buildname=${build_name} -e env=${environment.toLowerCase()} -e image_name_latest=${image_name_latest} -e service_name=${service_name} ./cloudrun/myproject/playbook.yml
        """ 
    }
    stage('remove docker image'){
        sh """
            docker rmi ${image_name}
            docker rmi ${image_name_latest}
        """
    }
}
  
