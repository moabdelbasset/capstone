node {
    def registry1 = 'moabdelhakim/capstone-blue'
    def registry2 = 'moabdelhakim/capstone-green'
    stage('Checking out git repo') {
      echo 'Checkout...'
      checkout scm
    }
    stage('Checking environment') {
      echo 'Checking environment...'
      sh 'git --version'
      echo "Branch: ${env.BRANCH_NAME}"
      sh 'docker -v'
    }
    stage("Linting") {
      echo 'Linting...'
      sh '/usr/bin/hlint blue/Dockerfile'
      sh '/usr/bin/hlint green/Dockerfile'
    }
    stage('Building image blue') {
	    echo 'Building Docker image blue...'
      withCredentials([usernamePassword(credentialsId: 'dockerhub', passwordVariable: 'dockerHubPassword', usernameVariable: 'dockerHubUser')]) {
	     	sh "sudo docker login -u ${env.dockerHubUser} -p ${env.dockerHubPassword} "
	     	sh "sudo docker build -t ${registry1} blue/."
	     	sh "sudo docker tag ${registry1} ${registry1}"
	     	sh "sudo docker push moabdelhakim/testblueimage"
      }
    }
    stage('Building image green') {
	    echo 'Building Docker image green...'
      withCredentials([usernamePassword(credentialsId: 'dockerhub', passwordVariable: 'dockerHubPassword', usernameVariable: 'dockerHubUser')]) {
	     	sh "sudo docker login -u ${env.dockerHubUser} -p ${env.dockerHubPassword}"
	     	sh "sudo docker build -t ${registry2} green/."
	     	sh "sudo docker tag ${registry2} ${registry2}"
	     	sh "sudo docker push ${registry2}"
      }
    }
    stage('Deploying to AWS EKS') {
      echo 'Deploying to AWS EKS...'
      dir ('./') {
        withAWS(credentials: 'aws.jenkins', region: 'us-east-2') {
            sh "aws eks --region us-east-2 update-kubeconfig --name ma-prod"
            sh "kubectl apply -f blue/blue-controller.json"
            sh "kubectl apply -f green/green-controller.json"
            sh "kubectl apply -f ./blue-green-service.json"
            sh "kubectl get nodes"
            sh "kubectl get pods"
        }
      }
    }
}
