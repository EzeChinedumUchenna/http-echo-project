#!/usr/bin/env groovy

pipeline {
    agent any

    environment {
        
        SONARSCANNER_HOME = tool 'sonarqube-scanner' // Tool name configured in Jenkins Global Tool Configuration
        MAX_ALLOWED_BUGS = 1
        JENKINS_API_TOKEN = credentials("JENKINS_API_TOKEN")
          }
 
    stages {

        stage('CLEAN WORKSPACE & CHECKOUT CODE') {
            steps {
                script {
                    // Clean workspace before checking out
                    deleteDir()

                    // Checkout the code from the GitHub repository
                    checkout([$class: 'GitSCM', branches: [[name: '*/main']], doGenerateSubmoduleConfigurations: false, extensions: [[$class: 'RelativeTargetDirectory', relativeTargetDir: 'http-echo-project']], submoduleCfg: [], userRemoteConfigs: [[url: 'https://github.com/EzeChinedumUchenna/http-echo-project.git']]])
                }
            }
        }

        stage('SONARQUBE SCAN') {
            steps {
                script {
                    // Use the configured SonarScanner installation
                    withSonarQubeEnv('sonarqube-server') {
                        sh """
                            ${SONARSCANNER_HOME}/bin/sonar-scanner \
                            -Dsonar.projectKey=http-echo-project \
                            -Dsonar.projectName="http-echo-project" \
                            -Dsonar.sources=. \
                            -Dsonar.go.coverage.reportPaths=coverage.out \
                        
                        """
                    }
                }
            }
        }

       /** stage('QUALITY GATE ANALYSIS') {
          steps {
            script {
             timeout(time: 3, unit:'MINUTES') {
             waitForQualityGate abortPipeline: true, credentialsId: 'Jenkins-sonaqube-Token'
          }
        }
     }
   } **/

        stage("BUILD IMAGE") {
            steps {
                script {
                    // Navigate to the directory containing the Dockerfile
                    dir('http-echo-project') {
                        // Build the Docker image
                        sh 'docker build -t anpauthuser.azurecr.io/http-echo-project:$BUILD_NUMBER .'
                  }
                }
            }
        }

        
    stage('PUSH TO AZURE CONTAINER REGISTRY') {
            steps {
                // Push the Docker image to Azure Container Registry
                script {
                    echo "deploying image to ACR ...."
                    withCredentials([usernamePassword(credentialsId: 'azure_acr', passwordVariable: 'PASS', usernameVariable: 'USER')]) {
        sh "echo $PASS | docker login -u $USER --password-stdin anpauthuser.azurecr.io"
        sh 'docker push anpauthuser.azurecr.io/http-echo-project:$BUILD_NUMBER'
                }
            }
        }
        //stage('Deploy to Azure') {
          //  steps {
                // Add your deployment steps here
                // This could include updating a Kubernetes deployment, triggering a release, etc.
                // Example: deploy to Azure Kubernetes Service (AKS)
          //      script {
                    // Use Azure CLI or Kubernetes CLI to update deployment
            //        sh "az aks update -n your-aks-cluster -g your-resource-group --image ${ACR_SERVER}/${DOCKER_IMAGE_NAME}:${env.BUILD_NUMBER}"
              //  }
            //}
        //}
    }
   stage('Clean Up Artifact') {
            steps {
                    script {
                        sh 'docker rmi anpauthuser.azurecr.io/http-echo-project:$BUILD_NUMBER'
                    }
            }
   }


        
  /** stage('Trigger CD pipeline') {
            steps {
                    script {
                       // sh "curl -v -k --user userman:${JEKINS_API} -X POST -H 'cache-control: no cache' -H token=TOKEN_NAME'
                        sh "curl -v -k --user chinedum:${JENKINS_API_TOKEN} -X POST -H 'cache-control: no-cache' -H 'content-type: application/x-www-form-urlencoded' --data 'BUILD_NUMBER=${BUILD_NUMBER}' '20.121.45.30:8080/job/emialApp-CD-Job/buildWithParameters?token=email_app_token'"
                    }
            }
   }  

} **/
     stage('TRIGGER CD PIPELINE') {
            steps {
                    script {
                      
                       withCredentials([usernamePassword(credentialsId: 'github_Credential', passwordVariable: 'PASS', usernameVariable: 'USER')]) {
                    // First we are going to attach a metadata to our commit. Like email and username, else Jenkins will complain. This is very important and a must-have at first commit but can be remove aftr that.
                        // Navigate into the 'http-echo-project' directory
                        dir('http-echo-project') {
                              sh "ls -al"
                            dir('helm-Chart') {
                              sh "ls -al"
                              sh 'git init . '
                              sh 'git config user.email "http-echo@gmail.com"' 
                              sh 'git config user.name "http-echo"'
                        // sh "sudo snap install helm --classic"
                        //sh "helm template . > kubernetes-template.yaml"
                        //sh "ls -al"
                        // sh "cat kubernetes-template.yaml"
                    // Note can set the above globally for all the project by adding '--global'
                    // sh 'git config --global user.email "nedum_jenkins@gmail.com"' 
                    // sh 'git config --global user.name "nedum_jenkins"' 
                    // Because my Github Password contain special character @, I will need to encode it else it wont work with Jenkins.
                            def encodedPassword = URLEncoder.encode(PASS, "UTF-8")

                        // Set the Git remote URL with the encoded password
                         sh "git remote -v | grep origin || git remote add origin https://${USER}:${PASS}@github.com/EzeChinedumUchenna/http-echo-project-CD "
                         sh "git remote set-url origin https://${USER}:${PASS}@github.com/EzeChinedumUchenna/http-echo-project-CD "
                        //sh 'git config pull.rebase true'
                        sh "git config pull.rebase true"
                        sh "git pull origin main"
                        sh "ls -al"
                        //sh 'git config pull.rebase false'
                        //sh 'git pull origin HEAD:refs/heads/main'
                        sh 'git fetch origin'
                        // sh 'git fetch origin HEAD:refs/heads/main'
                        // sh 'git merge origin/main main'
                        //sh "sed -i 's/nedumpythonapp:*/nedumpythonapp:${BUILD_NUMBER}/g' deployment.yaml"
                        //sh "sed -i 's/nedumpythonapp*/nedumpythonapp:${BUILD_NUMBER}/g' deployment.yaml"
                        sh "sed -i 's/http-echo-project.*/http-echo-project:${BUILD_NUMBER}/g' values.yaml"

                        sh "cat values.yaml"
                        sh 'git add .'
                        sh 'git commit -m "updated file"'
                        //sh 'git pull origin main --rebase'
                        sh 'git push origin HEAD:refs/heads/main' //here I want to push to main branch. Selete any branch you want to push to Eg sh 'git push origin HEAD:refs/heads/bug-fix'
                        //sh 'git push HEAD:main'
                       }
                      }
                    }
                }
            }
         } 
     }

    //PUSHING NOTIFICATION TO MY EMAIL. this will send email to me if the build fails or succeeds
   post {
        failure {
            script {
                mail (to: 'ezechinedum504@gmail.com',
                        subject: "Job '${env.JOB_NAME}' (${env.BUILD_NUMBER}) failed",
                        body: "Please visit ${env.BUILD_URL} for further information.",
                        attachmentsPattern: 'trivy.txt,trivyimage.txt'
                );
                }
            }
         success {
             script {
                mail (to: 'ezechinedum504@gmail.com',
                        subject: "Job '${env.JOB_NAME}' (${env.BUILD_NUMBER}) success.",
                        body: "Please visit ${env.BUILD_URL} for further information.",
                        attachmentsPattern: 'trivy.txt,trivyimage.txt'


                  );
                }
          }      
    } 
}
