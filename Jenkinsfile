#!groovy

def providerNamespace = 'ariga'
def providerId        = 'atlas'
def providerVersion   = 'v0.1.9-tivo.2'

pipeline {
    agent { label 'docker' }

    options {
        timeout( time: 1, unit: 'HOURS' )
        buildDiscarder( logRotator(daysToKeepStr: '15', numToKeepStr: '10') )
    }

    stages {
        stage('Build Application') {
            agent {
                dockerfile {
                    filename 'Dockerfile.tivo-build'
                    label 'docker'
                    reuseNode true
                }
            }
            steps {
                sh """
	            GOOS=linux  GOARCH=amd64 go build -o ./out/linux_amd64/terraform-provider-${providerId}_${providerVersion}
	            GOOS=linux  GOARCH=arm64 go build -o ./out/linux_arm64/terraform-provider-${providerId}_${providerVersion}
                GOOS=darwin GOARCH=amd64 go build -o ./out/darwin_amd64/terraform-provider-${providerId}_${providerVersion}
                GOOS=darwin GOARCH=arm64 go build -o ./out/darwin_arm64/terraform-provider-${providerId}_${providerVersion}
                """
            }
        }

        stage( 'Publish to Provider Registry' ) {
            when { branch 'tivo-build' }

            agent {
                docker {
                    image "${TERRAFORM_REGISTRY_PUBLISHING_IMAGE}"
		            alwaysPull true
                    label 'docker'
                    reuseNode true
                }
            }

            options { skipDefaultCheckout() }

            steps {
                withCredentials([usernamePassword(credentialsId: 'iam-terraform-provider-registry', usernameVariable: 'AWS_ACCESS_KEY_ID', passwordVariable: 'AWS_SECRET_ACCESS_KEY')]) {
                    sh "release.sh ${WORKSPACE} ${providerNamespace} ${providerId}"
                }
            }
        }
    }

    post {
        success {
            echo "Yay SUCCESS ..."
            script {
                def msg = "${currentBuild.result}: `${env.JOB_NAME}` #${env.BUILD_NUMBER}:\n${env.BUILD_URL}"
                //slackSend(channel: "#terraform-providers", color: 'good', message: msg)
            }
        }

        failure {
            echo "Boo FAILURE ..."
            script {
                def msg = "${currentBuild.result}: `${env.JOB_NAME}` #${env.BUILD_NUMBER}:\n${env.BUILD_URL}"
                //slackSend(channel: "#terraform-providers", color: 'danger', message: msg)
            }
        }
    }
}
