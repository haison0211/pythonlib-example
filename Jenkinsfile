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
      volumeMounts:
        - name: cache
          mountPath: /usr/local/lib/python3.11/site-packages
  volumes:
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
                    '''
                } 
                stage('Publish to artifactory') {
                    sh'''
                    ls dist
                    mv .pypirc ~
                    chmod 600 ~/.pypirc
                    python3 -m twine upload -r pypi dist/mypythonlib-0.1.0-py3-none-any.whl
                    '''
                }
            } 
    }
}