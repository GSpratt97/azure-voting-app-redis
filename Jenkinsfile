pipeline {
   agent any

   stages {
      stage('Verify Branch') {
         steps {
            echo "$GIT_BRANCH"
         }
      }
      stage('Docker Build') {
         steps {
            pwsh(script: 'docker images -a')
            pwsh(script: """
               cd azure-vote/
               docker images -a
               docker build -t jenkins-pipeline .
               docker images -a
               cd ..
            """)
         }
      }
      stage('Start test app') {
         steps {
            pwsh(script: """
               docker-compose up -d
               ./scripts/test_container.ps1
            """)
         }
         post {
            success {
               echo "App started successfully :)"
            }
            failure {
               echo "App failed to start :("
            }
         }
      }
      stage('Run Tests') {
         steps {
            pwsh(script: """
               pytest ./tests/test_sample.py
            """)
         }
      }
      stage('Stop test app') {
         steps {
            pwsh(script: """
               docker-compose down
            """)
         }
      }
      // stage('Push Container') {
      //    steps {
      //       echo "Workspace is $WORKSPACE"
      //       dir("$WORKSPACE/azure-vote") {
      //          script {
      //             docker.withRegistry('https://registry.hub.docker.com', 'DockerHub') {
      //                def image = docker.build('gspratt97/voting_app_pipeline_add-tests')
      //                image.push()
      //             }
      //          }
      //       }
      //    }
      // }
      // stage('Run Trivy') {
      //    steps {
      //       sh 'trivy --version'
      //       sh 'trivy -c'
      //       sh 'trivy gspratt97/voting_app_pipeline_add-tests'
      //    }
      // }
      // stage('Run Anchore') {
      //    steps {
      //       anchore name: 'anchore-cli'
      //    }
      // }
      stage('Container Scanning') {
         parallel {
            stage('Run Anchore') {
               steps {
                  pwsh(script: """
                  Write-Output "gspratt97/voting_app_pipeline_add-tests" > anchore_images
                  """)
                  anchore bailOnFail: false, bailOnPluginFail: false, name: 'anchore_images'
               }
            }
            stage('Run Trivy') {
               steps {
                  sleep(30) {
                     sh 'trivy --version'
                     sh 'trivy -c'
                     sh 'trivy gspratt97/voting_app_pipeline_add-tests'
                  }
               }
            }
         }
      }
   }
}
