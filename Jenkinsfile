pipeline {
    agent none
    options {
        skipStagesAfterUnstable()
    }
    stages {
        stage('Build') {
            agent {
                docker {
                    image 'python:3.12.0-alpine3.18'
                }
            }
            steps {
                sh 'python -m py_compile sources/add2vals.py sources/calc.py'
                stash(name: 'compiled-results', includes: 'sources/*.py*')
            }
        }
        stage('Test') {
            agent {
                docker {
                    image 'qnib/pytest'
                }
            }
            steps {
                sh 'py.test --junit-xml test-reports/results.xml sources/test_calc.py'
            }
            post {
                always {
                    junit 'test-reports/results.xml'
                }
            }
        }
        stage('Deliver') {
           agent any
               environment {
                   //VOLUME = '$(pwd)/sources:/src'
                   VOLUME = '$PWD/sources:/src'
                   IMAGE = 'cdrx/pyinstaller-linux:python3'
               }
               steps {
                   dir(path: env.BUILD_ID) {
                       unstash(name: 'compiled-results')
                     
                       //https://docs.python.org/3/distutils/builtdist.html
                       sh 'python3 setup.py bdist_dumb --format=zip'
                   }
               }
               post {
                   success {
                       archiveArtifacts "${env.BUILD_ID}/dist/*"
                   }
               }
        }
    }
}