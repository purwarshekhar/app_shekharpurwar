pipeline {
  agent any

  environment {
    scannerHome = 'sonar_scanner_dotnet'
    username = 'shekharpurwar'
    appName = 'Helloworld'
  }

  options {

    timestamps()

    //Set a timeout period for the Pipeline run, after which Jenkins should abort the Pipeline
    timeout(time: 1, unit: 'HOURS')

    buildDiscarder(logRotator(
      // number of build logs to keep
      numToKeepStr: '10',
      // history to keep in days
      daysToKeepStr: '30'
    ))
  }

  stages {

    stage('Git Checkout') {
      steps {
        git 'https://github.com/purwarshekhar/app-shekharpurwar.git'

      }
    }

    stage("nuget restore") {
      steps {

        echo "Deployment started for - ${BRANCH_NAME} branch"
        bat "dotnet restore"
      }
    }

    stage('Start sonarqube analysis') {
      when {
        branch "master"
      }

      steps {
        echo "Start sonarqube analysis step"
        withSonarQubeEnv('Test_Sonar') {
          bat "dotnet sonarscanner begin /k:sonar-${userName} /n:sonar-${userName} /v:1.0"
        }
      }
    }

    stage('Code build') {
      steps {
        //Cleans the output of a project
        echo "Clean Previous Build"
        bat "dotnet clean"

        //Builds the project and all of its dependencies
        echo "Code Build"
        bat 'dotnet build -c Release -o "ProductManagementApi/app/build"'
        bat 'dotnet test /p:CollectCoverage=true /p:CoverletOutputFormat=opencover -l:trx;LogFileName=ProductManagementApi.xml'
      }
    }

    stage('Stop sonarqube analysis') {
      when {
        branch "master"
      }

      steps {
        echo "Stop sonarqube analysis"
        withSonarQubeEnv('Test_Sonar') {
          bat "dotnet sonarscanner end"
        }
      }
    }

    stage("Release artifact") {
      when {
        branch "develop"
      }

      steps {
        echo "Release artifact step"
        bat "dotnet publish -c Release -o ${appName}/app/${userName}"
      }
    }

    // stage('Kubernetes Deployment') {
    // steps{
    // bat "kubectl apply -f deployment.yaml"
    // }
    //}
  }

  post {
    always {
      echo 'Workspace Cleanup'
      cleanWs()
    }
  }
}
