podTemplate(yaml:
    '''
apiVersion: v1
kind: Pod
spec:
  containers:
    - name: python
      image: 'python:alpine3.15'
      command:
        - sleep
      args:
        - 99d
    - name: docker
      image: docker:19.03.1
      command: ['sleep', '99d']
      env:
        - name: DOCKER_HOST
          value: tcp://localhost:2375
    - name: docker-daemon
      image: docker:19.03.1-dind
      env:
        - name: DOCKER_TLS_CERTDIR
          value: ""
      securityContext:
        privileged: true
      volumeMounts:
        - name: private-registries
          mountPath: /etc/docker/daemon.json
          subPath: daemon.json
  volumes:
    - name: private-registries
      configMap:
        name: docker-agent
'''
) {

    node(POD_LABEL) {
            container('python') {
                stage('Package a python library') {
                    checkout scm
                    sh ''' 
                    python3 -m venv venv
                    echo "INSTALL DEPENDENCY"
                    /usr/local/bin/python -m pip install --upgrade pip
                    pip install wheel
                    pip install setuptools
                    pip install twine
                    pip install pytest==4.4.1
                    pip install pytest-runner==4.4
                    echo "RUN TEST"
                    python setup.py pytest
                    echo "BUILD LIBRARY"
                    python setup.py bdist_wheel
                    ls dist
                    '''
                    
                }  
            } 
            container('docker')   {
                stage('Containerization') {
                    docker_image = docker.build("sonatype-nexus-nexus-repository-manager-docker-5000.nexus:5000/java-netty-app:v1")
                    withDockerRegistry(url: 'http://sonatype-nexus-nexus-repository-manager-docker-5000.nexus:5000', credentialsId: 'docker-registry-credential') {
                          docker_image.push()
                }  
            }
            }
    }
}