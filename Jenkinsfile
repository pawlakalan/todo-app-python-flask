pipeline {
    agent {label 'agent1'}
    options {
        skipDefaultCheckout(true)
    }
    stages {
        stage('Build') { 
            steps {
                cleanWs()
                checkout scm
                script {
                    sh """
                    python3 -m venv todo-app-venv
                    . todo-app-venv/bin/activate
                    pip3 install -r requirements.txt
                    """
                }
            }
        }
        stage('Test') {
            steps {
                script {
                    sh """
                    . todo-app-venv/bin/activate 
                    python3 -m unittest test_app
                    """
                }
            }
    }
    stage('Artifact Upload') {
            steps {
                script {
                    dockerImage = docker.build("pawlakalan/todo-app-python-flask:latest")
                    withDockerRegistry([ credentialsId: "dockerhub-credentials", url: "" ]) {
                        dockerImage.push()
                    }
                }
            }
        }
        
        stage('Deploy') {
            // when {
            //     branch 'master'
            // }
            steps {
                script {
                    withCredentials([string(credentialsId: 'render-token', variable: 'RENDER_TOKEN')]) {
                        httpRequest "https://api.render.com/deploy/${RENDER_TOKEN}&imgURL=docker.io%2Fpawlakalan%2Ftodo-app-python-flask%3Alatest"
                    }
                }
            }
        }
    }
    post {
        always {
            cleanWs(cleanWhenNotBuilt: false,
                    deleteDirs: true,
                    disableDeferredWipeout: true,
                    notFailBuild: true,
                    patterns: [[pattern: '.gitignore', type: 'INCLUDE'],
                               [pattern: '.propsfile', type: 'EXCLUDE']])
        }
    }
}
