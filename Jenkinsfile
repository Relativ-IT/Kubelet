pipeline {
  agent {
    label 'Linux && Buildah'
  }

  environment{
    KUBELET="v1.34.2"
    SHA_AMD64="7a0e3fa6a6afd29e7191b4fabd1cb994458804c8a5cf1b622da77bcd6773244d23bf7f9e0e4b0b6f1485f33dd6d8908ea7e1faad4ba6c02d9c2fba3ce99ba888"

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
