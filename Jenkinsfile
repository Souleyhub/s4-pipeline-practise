pipeline {
    agent {
     label ("node1 || node2 || node3 || node4 || node5 || branch || main || jenkins-node || docker-agent || jenkins-docker2 || preproduction || production") 
    }
    environment {
		DOCKERHUB_CREDENTIALS=credentials('dockerhub')
	}

    options {
       buildDiscarder(logRotator(numToKeepStr: '2'))
       disableConcurrentBuilds()
       timeout (time: 60, unit: 'MINUTES')
       timestamps()
    }
    
    stages {
       stage('Setup parameters') {
            steps {
                script {
                    properties([
                        parameters([
                        
                        choice(
                            choices: ['Dev', 'Sanbox', 'Prod'], 
                            name: 'Environment'
                               ),


                        string(
                            defaultValue: 's4user',
                            name: 'user',
			    description: 'Enter the image Tag to deploy',
                            trim: true
                            ),
                        
                        string(
                            defaultValue: 'v2.2.0',
                            name: 'DBTag',
			                description: 'Enter the image Tag to deploy',
                            trim: true
                            ),

                        string(
                            defaultValue: 'v2.2.0',
                            name: 'UITag',
			    description: 'Enter the image Tag to deploy',
                            trim: true
                            ),

                        string(
                            defaultValue: 'v2.2.0',
                            name: 'WEATHERTag',
			    description: 'Enter the image Tag to deploy',
                            trim: true
                            ),

                        
                        string(
                            defaultValue: 'v2.2.0',
                            name: 'AUTHTag',
			                description: 'Enter the image Tag to deploy',
                            trim: true
                        )
                        ])
                    ])
		}
	    }
       }
	    
       stage('permission') {
            steps {
                sh '''
		cat permission.txt | grep -o $USER
		echo $?
                '''
            }
        }
   
       stage('cleaning') {
            steps {
		 echo 'clean the environment'
            }
        }

        stage('SonarQube analysis') {
            agent {
                docker {
                  image 'sonarsource/sonar-scanner-cli:4.7.0'
                }
               }
               environment {
        CI = 'true'
        //  scannerHome = tool 'Sonar'
        scannerHome='/opt/sonar-scanner'
    }
            steps{
                withSonarQubeEnv('Sonar') {
                    sh "${scannerHome}/bin/sonar-scanner"
                }
            }
        }

        stage('build-dev') {
	     when{ 
          
          expression {
            env.Environment == 'Dev' }
	     }
            steps {
                sh '''
cd UI
docker build -t devopseasylearning2021/s4-ui:${BUILD_NUMBER}-$UITag .
cd -
cd DB
docker build -t devopseasylearning2021/s4-db:${BUILD_NUMBER}-$DBTag .
cd -
cd auth 
docker build -t devopseasylearning2021/s4-auth:${BUILD_NUMBER}-$AUTHTag .
cd -
cd weather 
docker build -t devopseasylearning2021/s4-weather:${BUILD_NUMBER}-$WEATHERTag .
cd -
	       '''
            }
        }

        stage('build-sanbox') {
		when{ 
          
          expression {
            env.Environment == 'sanbox' }
		}
            steps {
		sh '''
cd UI
docker build -t devopseasylearning2021/s4-ui:${BUILD_NUMBER}-$UITag .
cd -
cd DB
docker build -t devopseasylearning2021/s4-db:${BUILD_NUMBER}-$DBTag .
cd -
cd auth 
docker build -t devopseasylearning2021/s4-auth:${BUILD_NUMBER}-$AUTHTag .
cd -
cd weather 
docker build -t devopseasylearning2021/s4-weather:${BUILD_NUMBER}-$WEATHERTag .
cd -
		'''
                
            }
        }

        stage('build-prod') {
		when{ 
          
          expression {
            env.Environment == 'prod' }
		}
            steps {
                sh '''
cd UI
docker build -t devopseasylearning2021/s4-ui:${BUILD_NUMBER}-$UITag .
cd -
cd DB
docker build -t devopseasylearning2021/s4-db:${BUILD_NUMBER}-$DBTag .
cd -
cd auth 
docker build -t devopseasylearning2021/s4-auth:${BUILD_NUMBER}-$AUTHTag .
cd -
cd weather 
docker build -t devopseasylearning2021/s4-weather:${BUILD_NUMBER}-$WEATHERTag .
cd -
		'''
            }
        }

        stage('login') {
            steps {
		sh ''' 
                echo $DOCKERHUB_CREDENTIALS_PSW | docker login -u devopseasylearning2021 --password-stdin
		   '''
		 
            }
        }

        stage('push-to-dockerhub-dev') {
		when{ 
          
          expression {
            env.Environment == 'Dev' }
		}
            steps {
               sh '''
	       docker push devopseasylearning2021/s4-ui:${BUILD_NUMBER}-$UITag 
docker push devopseasylearning2021/s4-db:${BUILD_NUMBER}-$DBTag  
docker push devopseasylearning2021/s4-auth:${BUILD_NUMBER}-$AUTHTag  
docker push devopseasylearning2021/s4-weather:${BUILD_NUMBER}-$WEATHERTag
                  '''
            }
        }

        stage('push-to-dockerhub-sanbox') {
		when{ 
          
          expression {
            env.Environment == 'Sanbox' }
		}
            steps {
               sh '''
	       docker push devopseasylearning2021/s4-ui:${BUILD_NUMBER}-$UITag 
docker push devopseasylearning2021/s4-db:${BUILD_NUMBER}-$DBTag  
docker push devopseasylearning2021/s4-auth:${BUILD_NUMBER}-$AUTHTag  
docker push devopseasylearning2021/s4-weather:${BUILD_NUMBER}-$WEATHERTag
                  '''
            }
        }

        stage('push-to-dockerhub-prod') {
		when{ 
          
          expression {
            env.Environment == 'Prod' }
		}
            steps {
               sh '''
	       docker push devopseasylearning2021/s4-ui:${BUILD_NUMBER}-$UITag 
docker push devopseasylearning2021/s4-db:${BUILD_NUMBER}-$DBTag  
docker push devopseasylearning2021/s4-auth:${BUILD_NUMBER}-$AUTHTag  
docker push devopseasylearning2021/s4-weather:${BUILD_NUMBER}-$WEATHERTag
	          '''
            }
        }

        stage('update-helm-charts-dev') {

	      steps {
	        script {
	          withCredentials([
	            string(credentialsId: 'soultoken', variable: 'TOKEN'),
	            
	          ]) {

	            sh '''
	             git config --global user.name "souleyhub"
                     git config --global user.email "souleys15@gmail.com"
                rm -rf s4-pipeline-practise || true
                git clone https://souleyhub:$TOKEN@github.com/souleyhub/s4-pipeline-practise.git
                cd s4-pipeline-practise
		git checkout souleymane
 cat <<EOF > CHARTS/dev-values.yaml
         image:
           db:
              repository: devopseasylearning2021/s4-db
              tag: "${BUILD_NUMBER}-$DBTag"
           ui:
              repository: devopseasylearning2021/s4-ui
              tag: "${BUILD_NUMBER}-$UITag"
           auth:
              repository: devopseasylearning2021/s4-auth
              tag: "${BUILD_NUMBER}-$AUTHTag"
           weather:
             repository: devopseasylearning2021/s4-weather
             tag: "${BUILD_NUMBER}-$WEATHERTag"
EOF
    
	     git add -A
             git commit -m "testing jenkins"
             git push 
             
	     '''
                 
			 
	    
		}
              }
            }
       }


    stage('update-helm-charts-sanbox') {

	      steps {
	        script {
	          withCredentials([
	            string(credentialsId: 'soultoken', variable: 'TOKEN'),
	            
	          ]) {

	            sh '''
	             git config --global user.name "souleyhub"
                     git config --global user.email "souleys15@gmail.com"
                rm -rf s4-pipeline-practise || true
                git clone https://souleyhub:$TOKEN@github.com/souleyhub/s4-pipeline-practise.git
                cd s4-pipeline-practise/CHARTS
 cat <<EOF > sanbox-values.yaml
         image:
           db:
              repository: devopseasylearning2021/s4-db
              tag: "${BUILD_NUMBER}-$DBTag"
           ui:
              repository: devopseasylearning2021/s4-ui
              tag: "${BUILD_NUMBER}-$UITag"
           auth:
              repository: devopseasylearning2021/s4-auth
              tag: "${BUILD_NUMBER}-$AUTHTag"
           weather:
             repository: devopseasylearning2021/s4-weather
             tag: "${BUILD_NUMBER}-$WEATHERTag"
EOF               

                git add -A
                git commit -m "testing jenkins"
                git push https://souleyhub:$TOKEN@github.com/souleyhub/s4-pipeline-practise.git
                   '''
	            }
                }
              }
        }

	    
    stage('update-helm-charts-prod') {

	      steps {
	        script {
	          withCredentials([
	            string(credentialsId: 'soultoken', variable: 'TOKEN'),
	            
	          ]) {

	            sh '''
	             git config --global user.name "souleyhub"
                     git config --global user.email "souleys@gmail.com"
                rm -rf s4-pipeline-practise || true
                git clone https://souleyhub:$TOKEN@github.com/souleyhub/s4-pipeline-practise.git
                cd s4-pipeline-practise/CHARTS
cat <<EOF > prod-values.yaml
         image:
           db:
              repository: devopseasylearning2021/s4-db
              tag: "${BUILD_NUMBER}-$DBTag"
           ui:
              repository: devopseasylearning2021/s4-ui
              tag: "${BUILD_NUMBER}-$UITag"
           auth:
              repository: devopseasylearning2021/s4-auth
              tag: "${BUILD_NUMBER}-$AUTHTag"
           weather:
             repository: devopseasylearning2021/s4-weather
             tag: "${BUILD_NUMBER}-$WEATHERTag"
EOF               
                 
		git add -A
                git commit -m "testing jenkins"
                git push https://souleyhub:$TOKEN@github.com/souleyhub/s4-pipeline-practise.git}
                  '''
		    }
		}
              }
     } 
			  
	
			  
			  
     stage('wait for argocd') {
            steps {
                echo 'waiting for argocd to detect the image'
	    }
     }             
       
     
    stage('Hello') {
            steps {
                sh '''
                ls 
                pwd
                uname -r
		cat
		more
		less
                
                '''
            }
        }
   
    }	    
    
   post {
   
     success {
      slackSend (channel: '#development-alerts', color: 'good', message: "SUCCESSFUL: Application S4-EKTSS Job '${env.JOB_NAME} [${env.BUILD_NUMBER}]' (${env.BUILD_URL})")
     }

 
     unstable {
      slackSend (channel: '#development-alerts', color: 'warning', message: "UNSTABLE: Application S4-EKTSS Job '${env.JOB_NAME} [${env.BUILD_NUMBER}]' (${env.BUILD_URL})")
     }

     failure {
      slackSend (channel: '#development-alerts', color: '#FF0000', message: "FAILURE: Application S4-EKTSS Job '${env.JOB_NAME} [${env.BUILD_NUMBER}]' (${env.BUILD_URL})")
     }
   
     cleanup {
      deleteDir()
     }
   }
    
   }
