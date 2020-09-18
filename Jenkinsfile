pipeline {
  agent any
  stages {
    stage('Lint HTML') {
      steps {
        sh 'tidy -q -e *.html'
        sh '''$ sudo apt-get install xsltproc
$ git clone https://github.com/htacg/tidy-html5.git
$ cd tidy-html5
$ cd build/cmake
$ cmake ../.. -DCMAKE_BUILD_TYPE=Release -DBUILD_SHARED_LIB:BOOL=OFF
$ make
$ sudo make install'''
      }
    }

    stage('Lint Dockerfile') {
      steps {
        sh '''
						docker pull hadolint/hadolint:latest-debian
						pwd
						hadolint --ignore DL3006 Dockerfile
					'''
      }
    }

    stage('Build Docker Image') {
      steps {
        withCredentials(bindings: [[$class: 'UsernamePasswordMultiBinding', credentialsId: 'dockerhub', usernameVariable: 'DOCKER_USERNAME', passwordVariable: 'DOCKER_PASSWORD']]) {
          sh '''
						docker build --tag=microservice_capstone .
						docker image ls
					'''
        }

      }
    }

    stage('Push the Docker Image To Dockerhub') {
      steps {
        withCredentials(bindings: [[$class: 'UsernamePasswordMultiBinding', credentialsId: 'dockerhub', usernameVariable: 'DOCKER_USERNAME', passwordVariable: 'DOCKER_PASSWORD']]) {
          sh '''
						docker login -u $DOCKER_USERNAME -p $DOCKER_PASSWORD
						docker tag microservice_capstone mayanksingh1993/microservice_capstone
						docker push mayanksingh1993/microservice_capstone
					'''
        }

      }
    }

    stage('Set current kubectl context') {
      steps {
        withAWS(region: 'us-east-2', credentials: 'aws_credentials') {
          sh '''
							kubectl config use-context arn:aws:eks:us-east-2:297605572640:cluster/microservicesCluster
						'''
        }

      }
    }

    stage('Deploy blue container') {
      steps {
        withAWS(region: 'us-east-2', credentials: 'aws_credentials') {
          sh '''
							kubectl apply -f ./BlueController.json
						'''
        }

      }
    }

    stage('Deploy green container') {
      steps {
        withAWS(region: 'us-east-2', credentials: 'aws_credentials') {
          sh '''
							kubectl apply -f ./GreenController.json
						'''
        }

      }
    }

    stage('Create the service in the cluster, redirect to blue') {
      steps {
        withAWS(region: 'us-east-2', credentials: 'aws_credentials') {
          sh '''
							kubectl apply -f ./BlueService.json
						'''
        }

      }
    }

    stage('Wait user approval') {
      steps {
        input 'Ready to redirect the traffic to green?'
      }
    }

    stage('Create the service in the cluster, redirect to green') {
      steps {
        withAWS(region: 'us-east-2', credentials: 'aws_credentials') {
          sh '''
							kubectl apply -f ./GreenService.json
						'''
        }

      }
    }

  }
}