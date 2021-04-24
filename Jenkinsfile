pipeline {
    agent any
    options {
      timeout(unit: 'MINUTES', time: 60)
      timestamps()
      buildDiscarder logRotator(artifactDaysToKeepStr: '30', artifactNumToKeepStr: '50', daysToKeepStr: '30', numToKeepStr: '50')
    }
    environment {
        API_KEY = credentials('apikey')
    }
    stages {
        stage ('Artifactory Configuration') {
            steps {
                rtServer (
                    id: "T1i-Jfrog-Artifactory",
                    url: "http://192.168.1.222:8082/artifactory",
                    credentialsId: "jfrog_info"
                )
            }
        }
        stage('build .deb') {
            steps {
            echo "build .deb package"
            sh """
                rm -f *.deb
                fpm -s python -t deb  checkuser.py
            """
            }
        }
        stage ('Publish build info') {
            steps {
                rtBuildInfo (
                    captureEnv: true,
                    buildName: "${JOB_BASE_NAME}",
                    buildNumber: "${BUILD_NUMBER}"
                )
                rtUpload (
                    serverId: 'T1i-Jfrog-Artifactory',
                    spec: '''{
                        "files": [
                            {
                            "pattern": "*.deb",
                            "target": "t1i-deb-local/files/"
                            }
                        ]
                    }'''
                )
                rtPublishBuildInfo (
                    serverId: "T1i-Jfrog-Artifactory"
                )
            }
        }
    }
}
