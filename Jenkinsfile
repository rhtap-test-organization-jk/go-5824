library identifier: 'RHTAP_Jenkins@main', retriever: modernSCM(
  [$class: 'GitSCMSource',
   remote: 'https://github.com/redhat-appstudio/tssc-sample-jenkins.git'])


pipeline {
    agent {
        kubernetes {
            label 'jenkins-agent'
            cloud 'openshift'
            serviceAccount 'jenkins'
            podRetention onFailure()
            idleMinutes '30'
            containerTemplate {
                name 'jnlp'
                image 'image-registry.openshift-image-registry.svc:5000/jenkins/jenkins-agent-base:latest'
                ttyEnabled true
                args '${computer.jnlpmac} ${computer.name}'
            }
        }
    }
    environment {
        ROX_API_TOKEN = credentials('ROX_API_TOKEN')
        ROX_CENTRAL_ENDPOINT = credentials('ROX_CENTRAL_ENDPOINT')
        GITOPS_AUTH_PASSWORD = credentials('GITOPS_AUTH_PASSWORD')
        QUAY_IO_CREDS = credentials('QUAY_IO_CREDS')
    }
    stages {
        stage('init.sh') {
            steps {
                script {
                    rhtap.info ("Init")
                    rhtap.init()
                }
            }
        }
        stage('build') {
            steps {
                script {
                    rhtap.info( 'build_container..')
                    rhtap.buildah_rhtap()
                }
            }
        }
        stage('scan') {
            steps {
                script {
                    rhtap.info('acs_scans' )
                    rhtap.acs_deploy_check()
                    rhtap.acs_image_check()
                    rhtap.acs_image_scan()
                }
            }
        }
        stage('deploy') {
            steps {
                script {
                    rhtap.info('deploy' )
                    rhtap.update_deployment()
                }
            }
        }
        stage('summary') {
            steps {
                script {
                    rhtap.info('summary' )
                    rhtap.show_sbom_rhdh()
                    rhtap.summary()
                }
            }
        }
    }
}
