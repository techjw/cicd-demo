pipeline {
    agent any

    options {
        timeout(time: 20, unit: 'MINUTES')
    }

    environment{
        TEMPLATE_NAME = 'nginx-example'
        TEMPLATE_PATH = 'openshift/nginx-example'
        NGINX_VERSION = '1.20-ubi8'
        DEV_PROJECT = 'demo-jenkins-pipeline'
    }

    stages {
        stage('preamble') {
            steps {
                script {
                    openshift.withCluster() {
                        openshift.withProject(DEV_PROJECT) {
                            echo "Using project: ${openshift.project()}"
                        }
                    }
                }
            }
        }
        stage('cleanup') {
            steps {
                script {
                    openshift.withCluster() {
                        openshift.withProject(DEV_PROJECT) {
                            // delete everything with this template label
                            openshift.selector("all", [ template : TEMPLATE_NAME ]).delete()
                            // delete any secrets with this template label
                            if (openshift.selector("secrets", TEMPLATE_NAME).exists()) {
                                openshift.selector("secrets", TEMPLATE_NAME).delete()
                            }
                        }
                    }
                } // script
            } // steps
        } // stage
        stage('create') {
            steps {
                script {
                    openshift.withCluster() {
                        openshift.withProject(DEV_PROJECT) {
                            // create a new application from the templatePath
                            openshift.newApp("--template=${TEMPLATE_PATH}", "--name=${TEMPLATE_NAME}", "--param=NGINX_VERSION=${NGINX_VERSION}", "--labels=app.kubernetes.io/part-of=${TEMPLATE_NAME}")
                        }
                    }
                } // script
            } // steps
        } // stage
        stage('build') {
            steps {
                script {
                    openshift.withCluster() {
                        openshift.withProject(DEV_PROJECT) {
                            def builds = openshift.selector("bc", TEMPLATE_NAME).related('builds')
                            builds.untilEach(1) {
                                return (it.object().status.phase == "Complete")
                            }
                        }
                    }
                } // script
            } // steps
        } // stage
        stage('deploy') {
            steps {
                script {
                    openshift.withCluster() {
                        openshift.withProject(DEV_PROJECT) {
                            def rm = openshift.selector("dc", TEMPLATE_NAME).rollout()
                            openshift.selector("dc", TEMPLATE_NAME).related('pods').untilEach(1) {
                                return (it.object().status.phase == "Running")
                            }
                        }
                    }
                } // script
            } // steps
        } // stage
        stage('tag') {
            steps {
                script {
                    openshift.withCluster() {
                        openshift.withProject(DEV_PROJECT) {
                            // if everything else succeeded, tag the ${TEMPLATE_NAME}:latest image as ${TEMPLATE_NAME}-staging:latest
                            // a pipeline build config for the staging environment can watch for the ${TEMPLATE_NAME}-staging:latest
                            // image to change and then deploy it to the staging environment
                            openshift.tag("${TEMPLATE_NAME}:latest", "${TEMPLATE_NAME}-staging:latest")
                        }
                    }
                } // script
            } // steps
        } // stage
    } // stages
} // pipeline