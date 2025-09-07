pipeline {
    agent { 
        node {
            // label 'docker-agent-alpine'
            label 'docker-aws-cli'
        }
    }
    
    // triggers {
    //     pollSCM '* * * * *'
    // }

    parameters {
        booleanParam(name: 'DEVELOPMENT', defaultValue: false, description: 'Check to deploy to Development environment')
        booleanParam(name: 'PRODUCTION', defaultValue: false, description: 'Check to deploy to Production environment')
        booleanParam(name: 'DEVELOPMENT_ROLLBACK', defaultValue: false, description: 'Check to rollback for Development environment')
        booleanParam(name: 'PRODUCTION_ROLLBACK', defaultValue: false, description: 'Check to rollback for Production environment')
    }

    stages {
        stage('Clone Repository') {
            steps {
                echo 'Cloning repository'
                deleteDir()

                git branch: 'main',
                    url: 'https://github.com/sanalegon/energy-etl-stack.git'

                sh 'ls -lart'
            }
        }

        stage('Create CFN Template S3 Bucket') {
            steps {
                echo 'Creating S3 Bucket'
                sh 'ls'

                withCredentials([[$class: 'AmazonWebServicesCredentialsBinding', credentialsId: 'aws-crendentials-admin']]) {
                    dir('infrastructure') {
                        sh "aws cloudformation deploy --stack-name cfn-s3bucket --template-file ../create-template-bucket.yaml --region 'us-east-1' --capabilities CAPABILITY_IAM"
                        sh 'aws s3 cp ../ s3://dl-energy/ --recursive'
                    }
                }
            }
        }

        stage('Development Deployment') {
            steps {
                echo 'Deploying development services'

                script {
                    if (params.DEVELOPMENT) {
                        withCredentials([[$class: 'AmazonWebServicesCredentialsBinding', credentialsId: 'aws-crendentials-admin']]) {
                            dir('infrastructure/development/') {
                                sh 'echo "=================Development Deployment=================="'
                                sh "aws cloudformation deploy --stack-name EnergyDeployDevelopmentStack --template-file energy-stack.yaml --region 'us-east-1' --capabilities CAPABILITY_IAM CAPABILITY_NAMED_IAM"
                            }
                        }
                    }
                }
            }
        }

        // stage('Deploy') {
        //     steps {
        //         echo 'Deploying services...'
        //         withAWS(credentials: env.AWS_ACCESS_KEY, region: env.AWS_REGION) {
        //             sh './gradlew awsCfnMigrateStack awsCfnWaitStackComplete'
        //         }
        //     }
        // }
    }
}
