pipeline {
    agent any

    tools {
        maven 'maven'
    }

    environment {
        SONAR_TOKEN = credentials('sonar')
    }

    stages {

        stage('Checkout') {
            steps {

                echo 'Cloning GitHub Repo'

                git branch: 'main',
                url: 'https://github.com/sumapalla/CI-CD.git'
            }
        }

        stage('SonarQube Analysis') {
            steps {

                echo 'Scanning Project'

                sh 'ls -ltr'

                sh """
                    mvn sonar:sonar \
                    -Dsonar.host.url=http://localhost:9000 \
                    -Dsonar.token=$SONAR_TOKEN
                """
            }
        }

        stage('Build Artifact') {
            steps {

                echo 'Building Artifact'

                sh 'mvn clean package'
            }
        }

        stage('Docker Image Build') {
            steps {

                echo 'Building Docker Image'

                sh 'docker build -t katakamsuma/myapp:${BUILD_NUMBER} .'
            }
        }

        stage('Push to DockerHub') {
            steps {

                echo 'Pushing Docker Image'

                script {

                    withCredentials([string(credentialsId: 'docker', variable: 'dockerhub')]) {

                        sh 'docker login -u katakamsuma -p ${dockerhub}'

                        sh 'docker push katakamsuma/myapp:${BUILD_NUMBER}'
                    }
                }
            }
        }
stage('Update Deployment File') {

    environment {
        GIT_REPO_NAME = "CI-CD"
        GIT_USER_NAME = "sumapalla"
    }

    steps {
        echo 'Update Deployment File'

        withCredentials([string(credentialsId: 'githubtoken', variable: 'GITHUB_TOKEN')]) {

            sh '''
                git config --global user.email "sukuma.palla@gmail.com"
                git config --global user.name "Sukuma"

                sed -i "s/myapp:.*/myapp:${BUILD_NUMBER}/g" deploymentfiles/deployment.yml

                git add deploymentfiles/deployment.yml

                git commit -m "Update deployment image to version ${BUILD_NUMBER}" 

                git push https://${GIT_USER_NAME}:${GITHUB_TOKEN}@github.com/${GIT_USER_NAME}/${GIT_REPO_NAME}.git HEAD:main
            '''
        }
    }
}
    }
}
