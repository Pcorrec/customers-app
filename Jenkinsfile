pipeline {
    agent any

    stages {
        stage('GitHub') {
            steps {
                // Get some code from a GitHub repository
                git(
                    url: "https://github.com/Pcorrec/customers-app",
                    branch: "main",
                    changelog: true,
                    poll: true
                )
            }
        }

        stage('Build Artifact') {
            steps {
                bat "mvn clean package -DskipTests=true"
                archive 'target/*.jar' //so that they can be downloaded later
            }
        }

        stage('Tests - JUnit and Jacoco') {
            steps {
                parallel (
                    unitaires: {
                        bat "mvn test -Dgroups=unitaires"
                    },
                    integrations: {
                        bat "mvn test -Dgroups=integrations"
                    },
                    web: {
                        bat "mvn test -Dgroups=web"
                    }
                )
            }
            post {
                always {
                    junit 'target/surefire-reports/*.xml'
                    jacoco execPattern: 'target/jacoco.exec'
                }
            }
        }

        stage('Docker build') {
            steps {
                bat "docker build -t demo-devsecops ."
            }
        }

        stage('Docker run') {
            steps {
                bat 'docker run -d --rm -p 8180:8180 --name customers-app demo-devsecops'
            }
        }

        stage('Docker remove image') {
            steps {
                bat 'docker stop demo-devsecops'
                bat 'docker rmi demo-devsecops'
            }
        }
    }
}