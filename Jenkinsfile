pipeline {
    agent any

    environment {
        // You need to change region,access key and secret key of yours
        AWS_REGION = 'us-east-2'
        AWS_ACCESS_KEY_ID = credentials('your-aws-access-key-id')
        AWS_SECRET_ACCESS_KEY = credentials('your-aws-secret-access-key')
        APPLICATION_NAME = 'Nodejs'
        ENVIRONMENT_NAME = 'Nodejs-env'
        S3_BUCKET = 'elasticbeanstalk-us-east-2-006095271778' // Give your s3 name created by aws beanstalk
        SOURCE_BUNDLE_NAME = 'node_modules.zip'
    }

    stages {
       stage('Checkout') {
            steps {
                // Change the github repo into your
                checkout scmGit(branches: [[name: '*/main']], extensions: [], userRemoteConfigs: [[url: 'https://github.com/Vennilavan12/ebs.git']])
            }
        }

        stage('Build') {
            steps {
                script {
                    // Your build commands here
                    sh 'npm install'
                    sh 'npm test'
                }
            }
        }

        stage('Package') {
            steps {
                script {
                    // Package the application into a ZIP file
                    sh "zip -r $SOURCE_BUNDLE_NAME ."
                }
            }
        }

        stage('Deploy to Elastic Beanstalk') {
            steps {
                // Change the crendentialId into your id on jenkins credentials stored
                withCredentials([
                    [$class: 'AmazonWebServicesCredentialsBinding', credentialsId: 'AWS', accessKeyVariable: 'AWS_ACCESS_KEY_ID', secretKeyVariable: 'AWS_SECRET_ACCESS_KEY']
                ]) {
                    script {
                    // Deploy the application to Elastic Beanstalk
                        sh "aws s3 cp $SOURCE_BUNDLE_NAME s3://$S3_BUCKET/$SOURCE_BUNDLE_NAME"
                        sh "aws elasticbeanstalk create-application-version --application-name $APPLICATION_NAME --version-label $BUILD_NUMBER --source-bundle S3Bucket=$S3_BUCKET,S3Key=$SOURCE_BUNDLE_NAME"
                        sh "aws elasticbeanstalk update-environment --application-name $APPLICATION_NAME --environment-name $ENVIRONMENT_NAME --version-label $BUILD_NUMBER"
                    }   
                }
            }
        }
    }
}
