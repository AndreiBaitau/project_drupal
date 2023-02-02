pipeline {
      environment {
    registry = "andreibaitov/drupal"
    registryCredential = 'docker'

  }
      agent { label 'master'}
  stages {
      

     stage('Cloning git repositories') {
            parallel {
                stage('Clone docker repository') {
                    steps {
                        dir('docker') {
                            git url: 'https://github.com/AndreiBaitau/project_drupal.git', branch: 'drupal', credentialsId: 'git_cred'
                            sh 'ls -l'
                            
                           
                        }
                    }
                }
                stage('Clone drupal repository') {
                    steps {
                        dir('drupal') {
                            git url: 'https://github.com/AndreiBaitau/project_drupal.git', branch: 'master'
                            sh 'ls -l'
                            
                           
                        }
                    }
                }
                 stage('Clone argo repository') {
                    steps {
                        dir('argo') {
                            git url: 'https://github.com/AndreiBaitau/argo-cd.git', branch:'project'
                            sh 'ls -l'
                          
                        }
                    }
                }
                
            }
            
        }

    stage ("Lint dockerfile") {
       
        steps {
            sh 'hadolint /var/lib/jenkins/workspace/Drupal/docker/7/php8.0/apache-buster/Dockerfile | tee -a hadolint_lint.txt'
        }
        post {
            always {
                archiveArtifacts 'hadolint_lint.txt'
                            
            }

        }
    }
  stage('Building image') {
      steps{ 

        script {
             dockerImage = docker.build("$registry:1.$BUILD_NUMBER.1","./docker/7/php8.0/apache-buster/")
        }
      }
    }

    stage('Push Image to repo') {
      steps{
        script {
            docker.withRegistry( '', registryCredential ) {
            dockerImage.push()
          }
        }
      }
    }
    stage('Changing files') {
            parallel {
    stage('Changing tag in helm chart') {
      steps{
        script {
          sh" sed -i 's/1.*/1.$BUILD_NUMBER.1/' ./drupal/helm-source/drupal/Chart.yaml"
          
            
        }
        }
      }
    stage('Changing tag in argo yaml') {
      steps{
        script {
          sh" sed -i 's/targetRevision: 1.*/targetRevision: 1.$BUILD_NUMBER.1/' ./argo/project_drupal.yaml"
          
        }
        }}}
      }
      stage('Helm package') {
      steps{
        script {
          sh """
             cd drupal/helm-source
             helm package drupal
             mv drupal-1.$BUILD_NUMBER.1.tgz ../helm-releases
             cd ..
             helm repo index --url https://andreibaitau.github.io/project_drupal/ --merge index.yaml .
          """
          
        }
        }
      }
      
      stage('Push to drupal repo') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'git_cred', usernameVariable: 'USER', passwordVariable: 'PASS')]) {
                    script {
                        env.encodedPass=URLEncoder.encode(PASS, "UTF-8")
                    }
                    sh """
                        cd drupal
                        git config --global user.email "baitovandrei@gmail.com"
                        git config --global user.name "AndreiBaitau"
                        git add --all
                        git commit -m "Change files No.$BUILD_NUMBER"
                        git push https://${USER}:${encodedPass}@github.com/AndreiBaitau/project_drupal.git
                        sleep 5s
                    """
                }
            }
        }
      stage('Push to argo repo') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'git_cred', usernameVariable: 'USER', passwordVariable: 'PASS')]) {
                    script {
                        env.encodedPass=URLEncoder.encode(PASS, "UTF-8")
                    }
                    sh """
                        cd argo
                        git config --global user.email "baitovandrei@gmail.com"
                        git config --global user.name "AndreiBaitau"
                        git add --all
                        git commit -m "Change files No.$BUILD_NUMBER"
                        git push https://${USER}:${encodedPass}@github.com/AndreiBaitau/argo-cd.git
                        sleep 5s
                    """
                }
            }
        }  
      
      
  }
    post {
    success {
        withCredentials([string(credentialsId: 'token', variable: 'TOKEN'), string(credentialsId: 'chat', variable: 'CHAT_ID')]) {
        sh  ("""
            
            curl -s -X POST https://api.telegram.org/bot${TOKEN}/sendMessage -d chat_id=${CHAT_ID} -d parse_mode=markdown -d text='*Job name:* ${env.JOB_NAME}  \n\n*Build number:* ${env.BUILD_NUMBER} \n\n*Build status*: Success \n'
        """)
            
        }
      slackSend (color: '#00FF00', message: "\1 Build was successfuly done:\1 \n\n Job name --> ${env.JOB_NAME} \n Build number --> 1.${env.BUILD_NUMBER}.1")
    }
    failure {
         withCredentials([string(credentialsId: 'token', variable: 'TOKEN'), string(credentialsId: 'chat', variable: 'CHAT_ID')]) {
        sh  ("""
            
            curl -s -X POST https://api.telegram.org/bot${TOKEN}/sendMessage -d chat_id=${CHAT_ID} -d parse_mode=markdown -d text='*Job name:* ${env.JOB_NAME}  \n\n*Build number:* ${env.BUILD_NUMBER} \n\n*Build status*: Failure \n'
        """)
            
        }
      slackSend (color: '#FF0000', message: "Build was failed: \n\n Job name --> ${env.JOB_NAME} \n Build number --> 1.${env.BUILD_NUMBER}.1")
    }
  }

  }
