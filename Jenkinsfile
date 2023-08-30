pipeline {
  agent {
    kubernetes {
      yamlFile 'build-agent.yaml'
      defaultContainer 'maven'
      idleMinutes 1
    }

  }
  stages {
    stage('Build') {
      parallel {
        stage('Compile') {
          steps {
            container(name: 'maven') {
              sh 'mvn compile'
            }

          }
        }

      }
    }

    stage('Test') {
      parallel {
        stage('Unit Tests') {
          steps {
            container(name: 'maven') {
              catchError(buildResult: 'SUCCESS', stageResult: 'UNSTABLE') {
                sh 'mvn test'
              }
            }
          }
        }
        stage('Generate SBOM') {
          steps {
            container('maven') {
              sh 'mvn org.cyclonedx:cyclonedx-maven-plugin:makeAggregateBom'
            }
          }
          post {
            always {
              dependencyTrackPublisher projectName:'demo', projectVersion: '0.0.1', artifact: 'target/bom.xml', autoCreateProjects: true, synchronous: true
              archiveArtifacts allowEmptyArchive: true, artifacts: 'target/bom.xml', fingerprint: true, onlyIfSuccessful: false
            }
          }
        }
        stage('SCA') {
          steps {
            container('maven') {
              catchError(buildResult: 'SUCCESS', stageResult: 'FAILURE') {
                sh 'mvn org.owasp:dependency-check-maven:check'
              }
            }
          }
          post {
            always {
              archiveArtifacts allowEmptyArchive: true,
              artifacts: 'target/dependency-check-report.html',
              fingerprint: true, onlyIfSuccessful: true
              // dependencyCheckPublisher pattern: 'report.xml'
            }
          }
        }
        stage('OSS License Checker') {
          steps {
            container('licensefinder') {
              sh 'ls -al'
              sh '''#!/bin/bash --login
              /bin/bash --login
              rvm use default
              gem install license_finder
              license_finder
              '''
            }
          }
        }
      }
    }

    //stage('SAST') {
      //steps {
        //sh 'too long'
        //container('slscan') {
          //sh 'scan --type java,depscan --build'
        //}
      //}
      //post {
        //success {
          //archiveArtifacts allowEmptyArchive: true, artifacts: 'reports/*', fingerprint: true, onlyIfSuccessful: true
        //}
      //}
    //}

  stage('Package') {
    parallel {
      stage('Create Jarfile') {
        steps {
          container(name: 'maven') {
            sh 'mvn package -DskipTests'
          }
        }
      }

      stage('Docker BnP') {
        steps {
          container(name: 'kaniko') {
            //sh '''/kaniko/executor --verbosity debug -f `pwd`/Dockerfile -c `pwd` --insecure --skip-tls-verify --cache=true --destination=docker.io/shichoc/dso-demo:latest'''
            sh '''/kaniko/executor --verbosity debug -f `pwd`/Dockerfile -c `pwd` --insecure --skip-tls-verify --cache=true --destination=docker.io/jokerchuang/dso-demo:latest'''
          }
        }
      }
    }
  }

    stage('Deploy to Dev') {
      steps {
        sh 'echo done'
      }
    }
  }
}
