pipeline {
  agent {
    label 'Linux && Buildah'
  }

  environment{
    KUBELET="v1.35.2"
    SHA_AMD64="499a61b38dbcddb7105a89cac0f63e1d9a08178c051d693a8095de65939deb4e12ff4a11116d69ec69e14da0d929c8344bbb8ea8ad06cc65d749d327ee0c7c55"

    IMAGE = "kubelet"
    LOCAL_REGISTRY_IMAGE_LATEST_NAME = "${env.REGISTRY_LOCAL}/${env.IMAGE}:latest"
    LOCAL_REGISTRY_IMAGE_VERSION_NAME = "${env.REGISTRY_LOCAL}/${env.IMAGE}:${env.KUBELET}"
  }

  stages {
    stage('Initialize') {
      parallel {
        stage('Advertising start of build') {
          steps{
            slackSend color: "#4675b1", message: "${env.JOB_NAME} build #${env.BUILD_NUMBER} started :fire: (<${env.RUN_DISPLAY_URL}|Open>)"
          }
        }

        stage('Print environments variables') {
          steps {
            sh 'printenv | sort'
          }
        }

        stage('Print Buildah infos') {
          steps {
            sh '''
              buildah version
              buildah info
            '''
          }
        }
      }
    }

    stage('Building image') {
      steps {
        sh '''
          buildah build \
            --pull \
            --build-arg KUBELET=${KUBELET} \
            --build-arg SHA=${SHA_AMD64} \
            -t $LOCAL_REGISTRY_IMAGE_VERSION_NAME \
            -f ./Dockerfile.amd64 \
            .
        '''
        sh 'buildah tag $LOCAL_REGISTRY_IMAGE_VERSION_NAME $LOCAL_REGISTRY_IMAGE_LATEST_NAME' 
      }
    }

    stage("Push latest image to local registry") {
      steps {
        sh 'buildah push $LOCAL_REGISTRY_IMAGE_LATEST_NAME'
        sh 'buildah push $LOCAL_REGISTRY_IMAGE_VERSION_NAME'
      }
    }
  }

  post {
    success {
      slackSend color: "#4675b1", message: "${env.JOB_NAME} successfully built :blue_heart: !"
    }

    failure {
      slackSend color: "danger", message: "${env.JOB_NAME} build failed :poop: !"
    }
    
    cleanup {
      cleanWs()
    }
  }
}
