def commit_id
pipeline {
  agent any
   
    stages {
        stage('Preparation') {
            steps { 
                checkout scm
                sh "git rev-parse --short HEAD > .git/commit-id"
                script {                        
                    commit_id = readFile('.git/commit-id').trim()
                }
            }
        }


        stage('Image Build') {
            steps {
                echo 'Building....'
                sh "docker build -t 127.0.0.1:8085/wordpress:${commit_id} ."
                echo 'build complete'
            }
	}
	
	stage('Image Store'){
	    steps {
                echo 'login to nexus...'
                sh 'docker login -u admin -p admin http://127.0.0.1:8085'
                echo 'Login Succeeded'
                
 		echo 'Pushing to nexus....'
	 	sh "docker push 127.0.0.1:8085/wordpress:${commit_id}"
                echo 'image pushed to nexus'
            }
        }
        stage('Deploy') {
            steps {
                echo 'Deploying to Kubernetes'
                sh "sed -i -r 's|wordpress:latest|wordpress:${commit_id}|' wordpress-deployment.yaml"
                script {
                     kubernetesDeploy(configs: "wordpress-deployment.yaml", kubeconfigId: "new-minikube-config")
                       }
                echo 'deployment complete'
                
            }
        }
    }
}
