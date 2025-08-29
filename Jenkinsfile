pipeline {
  agent {
    label 'Linux && Buildah'
  }

  environment{
    KUBELET="v1.34.0"
    SHA_AMD64="93ae93af2d39bf00747b66f365781c64880b4ca235031a7ecae7a9d017e04df7ca925f8c005b1da49447cf64cb3f1ecc790db460e60cd1f98f34aae1434ad103"

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
