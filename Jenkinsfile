pipeline {
    agent { label 'worker1' }

    environment {
        AWS_ACCESS_KEY_ID = credentials('AWS_ACCESS_KEY_ID')
        AWS_SECRET_ACCESS_KEY = credentials('AWS_SECRET_ACCESS_KEY')
        AWS_DEFAULT_REGION = credentials('AWS_DEFAULT_REGION')
        CLUSTER_NAME = credentials('CLUSTER_NAME')
        REGISTRY = credentials('REGISTRY')
        SERVICE_NAME = 'wayabank-customer-webapp'
        VERSION = sh (script: 'git rev-parse HEAD', returnStdout: true).trim().take(10)
        NAMESPACE = "${env.GIT_BRANCH == 'production' ? 'production' : 'staging' }"
    }

    stages {
	
        /*stage('Security Scan') {
            steps {
                withSonarQubeEnv('Waya Sonar') {
                    sh 'mvn clean verify sonar:sonar -DskipTests -Dsonar.projectKey=WAYA-PAY-CHAT-2.0-reactjs-customer-web'
                }
            }
        }*/

        
        stage("Build") { 
            steps{
                script {

                    sh 'yes | cp -rf .env.example .env'
                    if (env.GIT_BRANCH == 'production') {
                        sh 'sed -i "s/domain//" .env'
                    } else {
                        sh 'sed -i "s/domain/staging./" .env'
                    }

                    sh '''
                        sudo npm install
                        sudo npm run build
                    '''
                    echo 'Build with Nodejs'
                }
            }   
        }

        stage('Image Build') {
            steps {
                script {
                    dockerImage = docker.build "${REGISTRY}/${SERVICE_NAME}:${VERSION}"
                }
            }
        }

        stage('LOGIN TO ECR') {
            steps {
                script {
                    sh "aws ecr get-login-password --region eu-west-2 | docker login --username AWS --password-stdin ${REGISTRY}"
                }
            }
        }

        stage('Pushing to ECR') {
            steps {
                script {
                    sh "docker push ${REGISTRY}/${SERVICE_NAME}:${VERSION}"
                }
            }
        }

        stage('Deploy to Production') {
            steps {
                script {
                    sh '''
                        chmod +x ./deploy.sh
                        ./deploy.sh
                        '''
                }
            }
        }
    }
}
