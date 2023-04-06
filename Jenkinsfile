/*
Reguires the Parameter Separator plugin for sane parameter organization

This Jenkinsfile is meant to be shared between Radical and the City of Richmond (COR).

This pipeline is meant to play nicely with Blue Ocean, but will run as a default multi-branch pipeline

Parameters will not be available until the first time this pipeline runs. After that, go into
the Jenkins job and set initial values and rerun the job

The docker image that will be produced will be configured to run in production mode, which
means that environment variables needs to be configured for the image on the docker host that
will be running the container.

Shared Parameters
    Required

        DEPLOYMENT_ENVIRONMENT - Deployment environment to build for. Sets the initial value of NODE_ENV in the image
                                 This should be a choice parameter that defaults to "production", with other options
                                 defined in the config.js for this project
        REPO_HOST - Repository host where images should be pushed to. Must be provided without a scheme
        REPO_NAME - Name of the repository where images will be pushed
        REPO_CREDS - Credentials for pushing images to the repository

    Optional

        COMMON_API_REPO - Package source for @myrichmond/api-common.
                          If this is not provided the build will use the one defined in the source code.

In addition to the paramters outlined above, this Jenkinsfile has different parameter requirements depending on who the build owner is.

City Of Richmond
    Required
    Optional



For the testing phase the following environmont variables needs to be set for the build if the DEPLOYMENT_TARGET is production.

This parameter is required regardles of how you provide the other parameters for test. It's needed to confirm that an instance of the
INTERNAL_API_BASEURL - BaseURL for the internal API

TEST_EVIRONMENT_FILE - Path to a file containing environment variables that will be set for running tests.

The folling environment variables must be defined in the file:

Optional
    NEWMAN_ENVIRONMENT - Name of the environment file to use for running Postman scripts. If this is not provided Postman scripts will not run

*/

//Build owner for this build
//Must be set as a global property in Manage Jenkins->Global Properties with name BUILD_OWNER
//If this is not set it will default to COR
def buildOwner = 'COR'

//Name of the node to run on
//Since we're sharing this file at the moment, we check for the BUILD_OWNER global variable and if it's Radical build on master
def nodeName = 'android-build-box'

pipeline {
    agent any

    environment {
        PROJECT_LOCATION = '.'
        MODULE = 'app'
        VARIANT = 'bitrise'
        KEYSTORE_TEST_FILENAME = 'MyRMobileKeystore.keystore'
        KEYSTORE_RELEASE_FILENAME = 'MyRMobileKeystore.keystore'
        MAPS_API_KEY = 'redacted'
    }

    stages {
        stage('Prepare environment') {
            steps {
                script {
                    echo 'Generating local.properties ...'

                    writeFile file: "${env.PROJECT_LOCATION}/local.properties", text: """
                    MAPS_API_KEY=${env.MAPS_API_KEY}
                    """
                }
            }
        }

        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Install missing Android tools') {
            steps {
                sh "${env.PROJECT_LOCATION}/gradlew --quiet androidDependencies"
            }
        }

        stage('Android Lint') {
            steps {
                sh "${env.PROJECT_LOCATION}/gradlew lint${env.VARIANT.capitalize()}"
            }
        }

        stage('Android Unit Test') {
            steps {
                sh "${env.PROJECT_LOCATION}/gradlew test${env.VARIANT.capitalize()}UnitTest"
            }
        }

        stage('Build APK') {
            steps {
                sh "${env.PROJECT_LOCATION}/gradlew assemble${env.VARIANT.capitalize()}"
            }
        }

        stage('Sign APK') {
            steps {
                // Add the required signing steps here. You can use the "sign-apk" plugin and its configuration.
                sh "echo 'Signing APK'"
            }
        }

        stage('Archive') {
            steps {
                archiveArtifacts artifacts: '**/build/outputs/apk/**/*.apk', fingerprint: true
            }
        }
    }

    post {
        always {
            cleanWs()
        }
    }
}
