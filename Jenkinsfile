pipeline {
    agent any
    environment {
        KUBECTL_PATH = "${env.WORKSPACE}/kubectl"
        PATH = "${env.PATH}:${KUBECTL_PATH}"
    }

    tools {
        maven 'Maven 3.9.9'
        jdk 'Java 21'
        dockerTool 'Docker'
    }
    parameters {
        // Define parameters for manual trigger of stages
        booleanParam(name: 'BUILD', defaultValue: true, description: 'Builds the spring boot app')
        booleanParam(name: 'DEPLOY_BLUE', defaultValue: false, description: 'Deploy Blue Version')
        booleanParam(name: 'DEPLOY_GREEN', defaultValue: false, description: 'Deploy Green Version')
        booleanParam(name: 'SWITCH_BLUE', defaultValue: false, description: 'Switch Traffic to Blue Version')
        booleanParam(name: 'SWITCH_GREEN', defaultValue: false, description: 'Switch Traffic to Green Version')
        booleanParam(name: 'CLEANUP_BLUE', defaultValue: false, description: 'Cleanup Blue Version')
        booleanParam(name: 'CLEANUP_GREEN', defaultValue: false, description: 'Cleanup Green Version')
    }
    stages {
        stage('Install kubectl') {
            steps {
                script {
                    // Download kubectl binary
                    sh 'curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl"'
                    // Make kubectl executable
                    sh 'chmod +x ./kubectl'
                }
            }
        }
        stage('Maven build') {
            when {
                beforeAgent true
                expression { return params.BUILD }
            }
            steps {
                sh 'mvn clean package'
            }
        }

// Skipped due to docker in docker issue
//         stage('Build the image') {
//             when {
//                 beforeAgent true
//                 expression { return params.BUILD }
//             }
//             steps {
//                 sh 'mvn spring-boot:build-image'
//             }
//         }

        stage('Deploy to Blue') {
            when {
                beforeAgent true
                expression { return params.DEPLOY_BLUE }
            }
            steps {

                sh '$KUBECTL_PATH apply -f k8s/app/deployment-blue.yaml'

            }
        }


        stage('Deploy to Green') {
            when {
                beforeAgent true
                expression { return params.DEPLOY_GREEN }
            }
            steps {

                sh '$KUBECTL_PATH apply -f k8s/app/deployment-green.yaml'
            }
        }


        stage('Switch to Blue') {
            when {
                beforeAgent true
                expression { return params.SWITCH_BLUE }
            }

            steps {
                 sh '''$KUBECTL_PATH patch service spring-boot-app -n app -p ' { "spec" : { "selector" : { "version" : "blue" } } }' '''
            }

        }


        stage('Switch to Green') {
            when {
                beforeAgent true
                expression { return params.SWITCH_GREEN }
            }
            steps {
                sh '''$KUBECTL_PATH patch service spring-boot-app -n app -p ' { "spec" : { "selector" : { "version" : "green" } } }' '''
            }
        }


        stage('Clean up Blue') {
            when {
                beforeAgent true
                expression { return params.CLEANUP_BLUE }
            }

            steps {
                sh '$KUBECTL_PATH delete -f k8s/app/deployment-blue.yaml'
            }
        }

        stage('Clean up Green') {
            when {
                beforeAgent true
                expression { return params.CLEANUP_GREEN }
            }
            steps {
                sh '$KUBECTL_PATH delete -f k8s/app/deployment-green.yaml'
            }
        }
    }

    post {
        success {
            echo 'Pipeline successfully completed!'
        }
        failure {
            echo 'Pipeline failed, please review the logs!'
        }
    }
}

