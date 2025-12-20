pipeline {
  agent {
    label 'Linux && Buildah'
  }

  environment{
    KUBELET="v1.35.0"
    SHA_AMD64="179278fecb65d246443f58cef00ca2f2a9d0ac6fbdb310994f0ac7fca249f7bdc1c79ea7f3e5455c1e2d2460f5447d006bfa579f97b502ee7034b2a1927f934a"

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
