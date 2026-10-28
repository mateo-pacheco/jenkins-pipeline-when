pipeline {
    agent any

    environment {
        WEBSERVER = 'Nginx'
        WEB_DIRECTORY = '/home/jenkins/app-web'
        VOLUME_NAME = 'app-web-data'
    }

    stages {
        stage('Prepare web application') {
            steps {
                echo 'Preparing web application...'

                sh '''
                    mkdir -p "$WEB_DIRECTORY"
                    find "$WEB_DIRECTORY" -mindepth 1 -maxdepth 1 \
                        -exec rm -rf -- {} +

                    cp -r web/. "$WEB_DIRECTORY/"

                    echo "Files copied:"
                    ls -la "$WEB_DIRECTORY"
                '''
            }
        }

        stage('Drop previous container') {
            steps {
                echo 'Removing previous container...'

                sh 'docker rm -f app-web || true'
            }
        }

        stage('Create Apache container') {
            when {
                environment name: 'WEBSERVER', value: 'Apache'
            }

            steps {
                echo 'Creating Apache container...'

                sh '''
                    docker run -dit \
                        --name app-web \
                        -p 9100:80 \
                        -v "$VOLUME_NAME":/usr/local/apache2/htdocs \
                        httpd:2.4
                '''
            }
        }

        stage('Create Nginx container') {
            when {
                environment name: 'WEBSERVER', value: 'Nginx'
            }

            steps {
                echo 'Creating Nginx container...'

                sh '''
                    docker run -dit \
                        --name app-web \
                        -p 9100:80 \
                        -v "$VOLUME_NAME":/usr/share/nginx/html \
                        nginx:stable
                '''
            }
        }

        stage('Verify deployment') {
            steps {
                echo 'Verifying deployment...'

                sh '''
                    docker ps --filter name=app-web

                    docker exec app-web sh -c '
                        if [ -d /usr/share/nginx/html ]; then
                            ls -la /usr/share/nginx/html
                        else
                            ls -la /usr/local/apache2/htdocs
                        fi
                    '
                '''
            }
        }
    }

    post {
        always {
            echo 'These steps are always executed'
        }

        success {
            echo 'The deployment has worked'

            archiveArtifacts(
                allowEmptyArchive: true,
                artifacts: 'web/**',
                followSymlinks: false
            )

            cleanWs()
        }

        failure {
            echo 'An error has occurred'
        }
    }
}
