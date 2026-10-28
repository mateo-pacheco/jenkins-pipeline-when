pipeline {
    agent any

    parameters {
       choice choices: ['Apache', 'Nginx'], name: 'WEBSERVER'
        choice choices: ['Development', 'Test','Production'], name: 'ENV'
    }

    stages {
        stage('Remove previous container') {
            steps {
                sh 'docker rm -f app-web || true'
            }
        }

        stage('Create Apache Dev') {
            when {
                allOf {           
                 environment name: 'WEBSERVER', value: 'Apache'
                 environment name: 'ENV', value: 'Development'                
                }
             }

            steps {
                sh '''
                    docker run -dit \
                      --name app-web \
                      -p 9100:80 \
                      httpd:2.4
                '''
            }
        }

        stage('Create Nginx Test') {
            when {
                allOf {           
                 environment name: 'WEBSERVER', value: 'Nginx'
                 environment name: 'ENV', value: 'Test'
            }
             }

            steps {
                sh '''
                    docker run -dit \
                      --name app-web \
                      -p 9100:80 \
                      nginx:stable
                '''
            }
        }

        stage('Create Nginx Produccion') {
            when {
                allOf {           
                 environment name: 'WEBSERVER', value: 'Nginx'
                 environment name: 'ENV', value: 'Production'
            }
             }

            steps {
                sh '''
                    docker run -dit \
                      --name app-web \
                      -p 9100:80 \
                      nginx:stable
                '''
            }
        }

        stage('Error message Nginx') {
           when {
                allOf {
                     environment name: 'WEBSERVER', value: 'Nginx'
                     environment name: 'ENV', value: 'Development'                       
                     }               
            }

            steps {
            echo 'Nginx cannot be deployed in Development...'
         
            }
        }

        //Error is Webserver is Apache and Env is Test or Production
         stage('Error message Apache') {
           when {
                allOf {
                    environment name: 'WEBSERVER', value: 'Apache'
                    anyOf {
                        environment name: 'ENV', value: 'Test'
                        environment name: 'ENV', value: 'Production'
                     }
                }
            }

            steps {
            echo 'Apache cannot be deployed in Test or Production environments...'
         
            }
        }

        stage('Deploy web application') {
            steps {
                script {
                    if (env.WEBSERVER == 'Nginx') {
                        sh '''
                            docker cp web/. app-web:/usr/share/nginx/html/
                        '''
                    } else {
                        sh '''
                            docker cp web/. app-web:/usr/local/apache2/htdocs/
                        '''
                    }
                }
            }
        }

        stage('Verify deployment') {
            steps {
                sh '''
                    docker exec app-web sh -c '
                        if [ -f /usr/share/nginx/html/index.html ]; then
                            ls -la /usr/share/nginx/html
                        elif [ -f /usr/local/apache2/htdocs/index.html ]; then
                            ls -la /usr/local/apache2/htdocs
                        else
                            echo "ERROR: index.html not found"
                            exit 1
                        fi
                    '
                '''
            }
        }
    }

    post {
        always {
            echo 'Pipeline finished'
        }

        success {
            echo 'Deployment successful'
            archiveArtifacts allowEmptyArchive: true, artifacts: 'web/**'
            cleanWs()
        }

        failure {
            echo 'Deployment failed'
        }
    }
}
