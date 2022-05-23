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
        - name: cache
          mountPath: /usr/local/lib/python3.11/site-packages
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
    - name: cache
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
                    twine upload --repository https://artifact.example.com/repository/ASAP-Python-2.7-Hosted/ boto3-1.9.76-py2.py3-none-any.whl
                    '''
                }  
            } 
    }
}