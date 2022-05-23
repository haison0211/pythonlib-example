podTemplate(yaml:
    '''
apiVersion: v1
kind: Pod
spec:
  containers:
    - name: python
      image: 'python:3.11.0b1-bullseye'
      command:
        - sleep
      args:
        - 99d
    - name: docker
      image: docker:19.03.1
      volumeMounts:
          - mountPath: /usr/local/lib/python3.11/site-packages
            name: python-cache
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
    - name: python-cache
          persistentVolumeClaim:
            claimName: python-cache
'''
) {

    node(POD_LABEL) {
            container('python') {
                stage('Package a python library') {
                    checkout scm
                    sh ''' 
                    python3 -m venv venv
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