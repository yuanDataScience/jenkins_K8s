pipeline {
    agent {
  // set K8s agent. namespace and secrets are defined
  // by the K8s cluster running Jenkins pipeline
  // defined as 'pipeline script from scm' in pipeline definition
  kubernetes {
    namespace 'jenkins'
        defaultContainer 'python-env'
    yaml """
apiVersion: v1
kind: Pod
spec:
  containers:
  - name: python-env
    image: python:3.10
    command: ["cat"]
    tty: true

  - name: kaniko
    image: gcr.io/kaniko-project/executor:debug
    command: ["cat"]
    tty: true
    volumeMounts:
    - name: registry-credentials
      mountPath: /kaniko/.docker # Kaniko automatically looks here for config.json
  
  volumes:
  - name: registry-credentials
    secret:
      secretName: regcred
      items:
      - key: .dockerconfigjson
        path: config.json
"""
        }
    }

    stages {
         
        stage('Unit Tests') {
            steps {
                container('python-env') {
                    sh 'echo "Running tests..."'
                    sh 'python3 --version'
                    sh 'which python3'
                    sh 'pip3 --version'
                }
            }
        }

        stage('Build & Push') {
            steps {
                container('kaniko') {
                    // Back to a clean, one-line command
                    sh '/kaniko/executor --context=dir://${WORKSPACE} --dockerfile=Dockerfile --destination=huangyuan2000/fastapi-demo:${env.BUILD_NUMBER}'
                }
            }
        }
    }
}