pipeline {

    agent any

    parameters {

        string(
            name: 'IMAGE_TAG',
            defaultValue: 'v1.0',
            description: 'Docker image tag to build'
        )

        choice(
            name: 'GIT_BRANCH',
            choices: ['master', 'develop', 'release'],
            description: 'Git branch to checkout'
        )

        booleanParam(
            name: 'RUN_TESTS',
            defaultValue: true,
            description: 'Run Maven tests?'
        )

        booleanParam(
            name: 'RUN_CONTAINER',
            defaultValue: true,
            description: 'Run Docker container?'
        )

        choice(
            name: 'DEPLOY_ENV',
            choices: ['DEV', 'QA', 'PROD'],
            description: 'Deployment environment'
        )
    }

    stages {

        stage('Display Parameters') {
            steps {
                echo "================ PARAMETER VALUES ================"
                echo "IMAGE_TAG      = ${params.IMAGE_TAG}"
                echo "GIT_BRANCH     = ${params.GIT_BRANCH}"
                echo "RUN_TESTS      = ${params.RUN_TESTS}"
                echo "RUN_CONTAINER  = ${params.RUN_CONTAINER}"
                echo "DEPLOY_ENV     = ${params.DEPLOY_ENV}"
                echo "==================================================="
            }
        }

        stage('Checkout') {
            steps {
                echo "Checking out branch: ${params.GIT_BRANCH}"

                checkout([
                    $class: 'GitSCM',
                    branches: [[name: "*/${params.GIT_BRANCH}"]],
                    userRemoteConfigs: [[
                        url: 'https://github.com/abybarath27-lgtm/springpetclinic.git'
                    ]]
                ])
            }
        }

        stage('Build') {
            steps {
                echo "Building application..."

                sh 'mvn clean package -DskipTests'
            }
        }

        stage('Test') {
            when {
                expression {
                    params.RUN_TESTS == true
                }
            }

            steps {
                echo "RUN_TESTS=true -> Running Maven tests"

                sh 'mvn test'
            }

            post {
                always {
                    junit allowEmptyResults: true,
                          testResults: 'target/surefire-reports/*.xml'
                }
            }
        }

        stage('Skip Tests') {
            when {
                expression {
                    params.RUN_TESTS == false
                }
            }

            steps {
                echo "RUN_TESTS=false -> Maven tests are skipped"
            }
        }

        stage('Docker Build') {
            steps {
                echo "Building Docker image..."
                echo "Docker image: simple-java-maven-app:${params.IMAGE_TAG}"

                sh """
                    docker build \
                    -t simple-java-maven-app:${params.IMAGE_TAG} .
                """
            }
        }

        stage('Deploy DEV') {
            when {
                allOf {
                    expression {
                        params.DEPLOY_ENV == 'DEV'
                    }
                    expression {
                        params.RUN_CONTAINER == true
                    }
                }
            }

            steps {
                echo "Deploying to DEV environment"

                sh """
                    docker rm -f simple-java-maven-app-dev || true
                    docker run -d \
                    --name simple-java-maven-app-dev \
                    -p 8081:8080 \
                    simple-java-maven-app:${params.IMAGE_TAG}
                """
            }
        }

        stage('Deploy QA') {
            when {
                allOf {
                    expression {
                        params.DEPLOY_ENV == 'QA'
                    }
                    expression {
                        params.RUN_CONTAINER == true
                    }
                }
            }

            steps {
                echo "Deploying to QA environment"

                sh """
                    docker rm -f simple-java-maven-app-qa || true
                    docker run -d \
                    --name simple-java-maven-app-qa \
                    -p 8081:8080 \
                    simple-java-maven-app:${params.IMAGE_TAG}
                """
            }
        }

        stage('Deploy PROD') {
            when {
                allOf {
                    expression {
                        params.DEPLOY_ENV == 'PROD'
                    }
                    expression {
                        params.RUN_CONTAINER == true
                    }
                }
            }

            steps {
                echo "Deploying to PROD environment"

                sh """
                    docker rm -f simple-java-maven-app-prod || true
                    docker run -d \
                    --name simple-java-maven-app-prod \
                    -p 8082:8080 \
                    simple-java-maven-app:${params.IMAGE_TAG}
                """
            }
        }

        stage('Container Skipped') {
            when {
                expression {
                    params.RUN_CONTAINER == false
                }
            }

            steps {
                echo "RUN_CONTAINER=false -> Docker container deployment skipped"
            }
        }
    }

    post {

        success {
            echo "==================================================="
            echo "Pipeline completed successfully"
            echo "==================================================="
        }

        failure {
            echo "Pipeline failed"
        }
    }
}
