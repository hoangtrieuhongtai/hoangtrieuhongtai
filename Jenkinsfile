pipeline {
    agent any
    options {
      timeout(unit: 'MINUTES', time: 60)
      timestamps()
      buildDiscarder logRotator(artifactDaysToKeepStr: '30', artifactNumToKeepStr: '50', daysToKeepStr: '30', numToKeepStr: '50')
    }
    environment {
        API_KEY = credentials('API KEY')
    }
    stages {
        stage ('Artifactory Configuration') {
            steps {
                rtServer (
                    id: 'T1i-Jfrog-Artifactory',
                    url: 'http://10.9.14.4:8082/artifactory',
                    credentialsId: 'jfrog_info'
                )
            }
        }
        stage('build .deb') {
            steps {
            echo "build .deb package"
            sh """
                rm -f *.deb
                fpm -s gem -t deb nginx
                mv *.deb nginx-${BUILD_NUMBER}.deb
            """
            }
        }
        stage ('Publish build info and artifact') {
            steps {
                rtBuildInfo (
                    captureEnv: true,
                    buildNumber: '${BUILD_NUMBER}'
                )
                rtUpload (
                    serverId: 'T1i-Jfrog-Artifactory',
                    spec: '''{
                        "files": [
                            {
                            "pattern": "*.deb",
                            "target": "t1i-deb-local"
                            }
                        ]
                    }'''
                )
                rtPublishBuildInfo (
                    serverId: "T1i-Jfrog-Artifactory"
                )
            }
        }
        stage ('Issues Collection') {
            steps {
                rtCollectIssues (
                    serverId: 'T1i-Jfrog-Artifactory',
                    config: """{
                        "version": 1,
                        "issues": {
                            "trackerName": "JIRA",
                            "regexp": "(.+-[0-9]+)\\s-\\s(.+)",
                            "keyGroupIndex": 1,
                            "summaryGroupIndex": 2,
                            "trackerUrl": "https://bitbucket.surfcrew.com/projects/SAFLY/repos/devtools/commits/2df197abdc1b58a343767561e34ecff860b574ad",
                            "aggregate": "true",
                            "aggregationStatus": "RELEASED"
                        }
                    }"""
                )
            }
        }
    }
    post {
        always {
            echo "Build Completed!"
        }
        cleanup {
            print "Cleaning up workspace directories"
            cleanWs deleteDirs: true
        }
    }
}
