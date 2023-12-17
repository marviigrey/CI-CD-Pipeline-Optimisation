pipeline {
  agent any

  environment {
    GITHUB_REPO = 'ella-adeka/CI-CD-Pipeline-Optimisation'
    GITHUB_TOKEN = credentials('github-personal-access-token')
  }

  stages {

    stage('Checkout') {
      // checkout source code from version control
      steps {
        checkout scm
      }
    }

    // Build Stage
    stage('Build Image') {
      steps {
        script {
          bat 'echo "Building application..."'
          // Build for local/dev environment
          bat 'docker build -t ella-adeka/ci-cd-pipeline ./my_app'
        }
      }
    }

    // Test Stage
    stage('Test Dev') {
      //  Run Unit and Integration Tests
      steps {
        script {
          bat 'echo "Testing application..."'
          bat 'python -m pytest'
        }
      }
    }

    // Push Image to Dockerhub
    stage("Push Image to DockerHub") {
      steps {
        // Deploy application
        script {
          bat 'echo "Deploying application to dockerhub..."'
          withCredentials([string(credentialsId: 'dockerhubsecrets', variable: 'dockerhubpwd')]) {
            // log in to Docker hub
            bat "docker login -u ellaadeka -p ${dockerhubpwd}"
          }
          // Push image to dockerhub
          bat 'docker tag ci-cd-pipeline ellaadeka/ci-cd-pipeline'
          bat 'docker push ellaadeka/ci-cd-pipeline'
        }
      }
    }

    stage("Infrastructure Provisioning for Dev Env") {
      steps {
        script {
          // Initialise Terraform
          bat 'terraform init -backend-config="dev-backend.conf"'
          // Apply Terraform
          bat 'terraform apply -auto-approve -input=false'
        }
      }
    }

    stage("Testing  Staging Env") {
      steps {
        script {
          // bat 'act -j testing-staging'
          bat 'echo "Performing Dry run for testing...."'
          
        }
      }
    }

    stage("Infrastruture Provisioning for Staging Env") {
      steps {
        script {
          bat 'echo "Performing Dry run for deploying to staging env...."'
          // Initialise Terraform
          bat 'terraform init -reconfigure -backend-config="stag-backend.conf"'
          // Apply Terraform
          bat 'terraform apply -auto-approve -input=false'
        }
      }
    }

    stage("Infrastruture Provisioning for Production Env") {
      steps {
        script {
          bat 'echo "Performing dry run for deploying to production...."'
          // Initialise Terraform
          bat 'terraform init -reconfigure -backend-config="prod-backend.conf"'
          // Apply Terraform
          bat 'terraform apply -auto-approve -input=false'
        }
      }
    }
  }

  post {
    always {
      // Cleanup actions
      echo 'Always executing cleanup...'
    }
    
    success{
      // Actions to be performed on successful execution
      echo 'Pipeline succeeded! Notifying Github of release'
      emailext (
        subject: "SUCCESSFUL: Job '${env.JOB_NAME} [${env.BUILD_NUMBER}]'",
        body: """<p>SUCCESSFUL: Job '${env.JOB_NAME} [${env.BUILD_NUMBER}]':<p>Check console output at &QUOT;<a href='${env.BUILD_URL}'>${env.JOB_NAME} [${env.BUILD_NUMBER}]</a>&QUOT;</p>""",
        to: "dassalotbro1@gmail.com"
      )
    }
    
    failure{
      // Actions to be performed on pipeline failure
      echo 'Pipeline failed. Please review the build logs and fix the issues.'
      emailext (
        subject: "FAILED: Job '${env.JOB_NAME} [${env.BUILD_NUMBER}]'",
        body: """<p>FAILED: Job '${env.JOB_NAME} [${env.BUILD_NUMBER}]':<p>Check console output at &QUOT;<a href='${env.BUILD_URL}'>${env.JOB_NAME} [${env.BUILD_NUMBER}]</a>&QUOT;</p>""",
        to: "dassalotbro1@gmail.com"
      )
    }
  }
}
