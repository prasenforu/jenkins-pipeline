node{
  stage('Git Checkout'){
      git url: 'https://github.com/prasenforu/dockerapp',
          branch:'master'
  }
  stage('Build Docker Image'){
    sh 'docker build -t 10.90.1.78/ocp-dev/my-app:0.0.1 .'
  }
  stage('Test Image upload to Harbor Registry'){
    withCredentials([string(credentialsId: 'harbor-passwd', variable: 'harborRegistryPwd')]) {
       sh "docker login 10.90.1.78 -u prasen -p ${harborRegistryPwd}"
    }
    sh 'docker push 10.90.1.78/ocp-dev/my-app:0.0.1'
  }
  stage('Docker Image Scan'){
    sh '''
        export CLAIR_IMAGE=10.90.1.78/ocp-dev/my-app:0.0.1
        export CLAIR_HOST=http://10.90.1.78:6060

        HIGH=$(REGISTRY_INSECURE=true CLAIR_ADDR=$CLAIR_HOST /usr/local/bin/klar $CLAIR_IMAGE | tail -n 7 | grep High | awk \'{print$2}\')
        echo $HIGH

        if [[ $HIGH < 1 ]]; then
          echo "+++Image $CLAIR_IMAGE has passed scan threshold+++"
          docker tag 10.90.1.78/ocp-dev/my-app:0.0.1 10.90.1.78/ocp-dev/my-app:latest
        else
          echo "---Image $CLAIR_IMAGE has failed scan threshold+++"
          docker rmi 10.90.1.78/ocp-dev/my-app:0.0.1
          exit 1
        fi'''
  }
  stage('Latest Image upload to Harbor Registry'){
    withCredentials([string(credentialsId: 'harbor-passwd', variable: 'harborRegistryPwd')]) {
       sh "docker login 10.90.1.78 -u prasen -p ${harborRegistryPwd}"
    }
    sh 'docker push 10.90.1.78/ocp-dev/my-app:latest'
  }
}
