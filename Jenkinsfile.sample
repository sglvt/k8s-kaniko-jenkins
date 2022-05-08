pipeline {
    agent { label 'kaniko'}

    stages {
        stage('Build and push to registry') {
            steps {
                container('kaniko') {
                    sh '''executor \
                          --destination=docker.io/somerepo/kaniko-test:$(date -u +%Y-%m-%dT%H%M%S) \
                          --context=git://somegitrepo.com/example/sample-go-http-app.git#refs/heads/master
                    '''
                }
            }
        }
    }
}