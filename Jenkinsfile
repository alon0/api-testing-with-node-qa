pipeline {
  environment {
    DOCKERHUB_CREDENTIALS=credentials('dockerHub')
  }
  agent {
    kubernetes {
      // label 'jenkins-agent'
      defaultContainer 'jnlp'
      yaml '''
        apiVersion: v1
        kind: Pod
        metadata:
        labels:
          component: ci
        spec:
          # Use service account that can deploy to all namespaces
          serviceAccountName: jenkins
          containers:
          - name: docker
            image: docker:latest
            tty: true
            volumeMounts:
            - mountPath: /var/run/docker.sock
              name: docker-sock
            - mountPath: /usr/bin/docker
              name: docker-bin
          - name: test
            image: node
            tty: true
          volumes:
            - name: docker-sock
              hostPath:
                path: /var/run/docker.sock
            - name: docker-bin
              hostPath:
                path: /usr/bin/docker
      '''
    }
  }
  stages {
    stage('Execute QA Tests') {
      steps {
        container('test') {
          sh '''
            export BACKEND_API=${BACKEND_API}
            npm install -g mocha chai
            make test
          '''
        }
      }
    }
    stage('Promote to Stable') {
      steps {
        container('docker') {
          git credentialsId: 'git',
              branch: '${GIT_COMMIT_SHORT}',
              url: 'git@github.com:alon0/DevOps-proj.git' 
          sh '''
            docker build -t alon0/devops-proj:stable-${GIT_COMMIT_SHORT} .
            echo $DOCKERHUB_CREDENTIALS_PSW | docker login -u $DOCKERHUB_CREDENTIALS_USR --password-stdin
            docker push alon0/devops-proj:stable-${GIT_COMMIT_SHORT}
          ''' 
            }
        }
    }
  post {
    success {
        echo 'QA Passed'
        // build job: 'api-testing-with-node-qa', parameters: [string(name: 'BACKEND_API', value: '${BACKEND_API}')]
    }
  }
}
