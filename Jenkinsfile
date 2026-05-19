pipeline {
    agent {
  // set K8s agent. namespace and secrets are defined
  // by the K8s cluster running Jenkins pipeline
  // defined as 'pipeline script from scm' in pipeline definition
  kubernetes {
    namespace 'jenkins'
        defaultContainer 'python'
    yaml """
apiVersion: v1
kind: Pod
spec:
  containers:
    - name: python
      image:  huangyuan2000/python-dvc:3.10.1
      command:
        - sleep
        - infinity
      env:
        - name: AWS_ACCESS_KEY_ID
          valueFrom:
            secretKeyRef:
              name: minio-credentials
              key: access_key
        - name: AWS_SECRET_ACCESS_KEY
          valueFrom:
            secretKeyRef:
              name: minio-credentials
              key: secret_key
"""
  }
}

    environment {
            VENV_DIR = 'venv'
        }


    triggers {
        pollSCM('H 5 * * *') // Polls daily at 5 AM
    }

    stages{
        stage('Verify Python') {
            steps {
                sh 'python3 --version'
                sh 'which python3'
                sh 'pip3 --version'
            }
        }


        stage('Verify MinIO Credentials (Env)') {
            steps {
                sh '''
                    [ -n "$AWS_ACCESS_KEY_ID" ] || exit 1
                    [ -n "$AWS_SECRET_ACCESS_KEY" ] || exit 1
                    echo "MinIO credentials present"
                '''
            }
        }

        stage('Verify MinIO Access (mc)') {
            steps {
                sh '''
                    mc alias set minio http://minio.mlops.svc.cluster.local:9000 \
                        "$AWS_ACCESS_KEY_ID" "$AWS_SECRET_ACCESS_KEY"

                    echo "MinIO buckets:"
                    mc ls minio
                '''
            }
        }

        
stage('Verify DVC Configuration') {
            steps {
                sh '''
                    dvc --version
                    dvc root
                    dvc remote list
                    dvc remote default
                    echo "DVC configuration valid"
                '''
            }
        }

stage('Verify DVC Access to MinIO') {
    steps {
        sh '''
            echo "Checking DVC ↔ MinIO connectivity (read-only)"
            dvc status -c
            echo "DVC remote is reachable"
        '''
    }
}
        // stage('Setup Python Environment') {
        //     steps {
        //         sh '''
                                       
        //             python3 -m venv $VENV_DIR
        //             . $VENV_DIR/bin/activate

        //            pip install -r requirements.txt
        //         '''
        //     }
        // }



        // stage('Run Tests') {
        //     steps {
        //         sh '''
        //             . venv/bin/activate
        //             which pytest
        //             export PYTHONPATH=$(pwd)
        //             pytest
        //         '''
        //     }
        // }


    }
}