pipeline {

  environment {
    PROJECT = "new-life-experience-platform"
    APP_NAME = "gceme"
    IMAGE_TAG = "asia.gcr.io/${PROJECT}/${APP_NAME}:${BUILD_NUMBER}"
    JENKINS_CRED = "${PROJECT}"
  }
  
  agent {
    kubernetes {
      label 'jenkins-worker'
      defaultContainer 'jnlp'
      yaml """
        apiVersion: v1
        kind: Pod
        metadata:
        labels:
          component: ci
        spec:
          # Use service account that can deploy to all namespaces
          serviceAccountName: cd-jenkins
          strategy:
            type: Recreate
          containers:
            - name: golang
              image: golang:1.10
              command:
              - cat
              tty: true
            - name: gcloud
              image: asia.gcr.io/new-life-experience-platform/gcloud-docker:alpine
              command:
              - cat
              tty: true
              securityContext:
                privileged: true
              volumeMounts:
              - name: docker-socket-volume
                mountPath: /var/run/docker.sock
          volumes:
            - name: docker-socket-volume
              hostPath: 
                path: /var/run/docker.sock
                type: File
      """
    }
  }

  stages {
    stage('Test') {
      steps {
        container('golang') {
          sh '''
          ln -s `pwd` /go/src/sample-app
            cd /go/src/sample-app
            go test
          '''
        }
      }
    }
    stage('Build and Push') {
      steps {
        container('gcloud') {
          sh '''
            gcloud auth configure-docker asia.gcr.io
            gcloud docker --verbosity=error -- build -t ${IMAGE_TAG} . 
            gcloud docker --verbosity=error -- push ${IMAGE_TAG} 
          '''
        }
      }
    }
  }
}
