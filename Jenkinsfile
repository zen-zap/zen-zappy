pipeline {
    agent any

    environment {
        DOCKER_CREDENTIALS_ID = 'docker-credentials'
        NEXUS_CREDENTIALS_ID = 'nexus'
        SONARQUBE_CREDENTIALS_ID = 'sonar'
        SONARQUBE_URL = 'http://54.152.193.254:9000/'
        NEXUS_URL = '3.91.192.245:8081'
        NEXUS_REPOSITORY = 'circad' // http://3.91.192.245:8081/repository/circad/
        SONARQUBE_TOKEN = 'squ_040d4029f2a245903f5a3eefe8376dafa0fdfd59'
    }

    triggers {
        // Commented out since GitHub webhook is not used
        githubPush()
    }

    tools {
        maven 'Maven 3.8.7' // Use the Maven tool configured in Jenkins
        // dockerTool 'docker'
    }
    stages {
        stage('Checkout') {
            steps {
                git branch: 'main', url: 'https://github.com/zen-zap/zen-zappy.git'
            }
        }

        stage('Build') {
            steps {
                dir('maven-app/my-app') {
                    sh 'mvn clean package'
                }
            }
        }

        stage('SonarQube Analysis') {
            steps {
                dir('maven-app/my-app') {
                    withSonarQubeEnv('SonarQube') {
                        sh 'mvn sonar:sonar -Dsonar.projectKey=your-project-key -Dsonar.host.url=$SONARQUBE_URL -Dsonar.login=$SONARQUBE_TOKEN'
                    }
                }
            }
        }

        stage('Publish to Nexus') {
            steps {
                script {
                    dir('maven-app/my-app') {
                        def pom = readMavenPom file: 'pom.xml'
                        def artifactId = pom.artifactId
                        def version = pom.version
                        def packaging = pom.packaging
                        def file = "target/${artifactId}-${version}.${packaging}"
                        nexusArtifactUploader(
                            nexusVersion: 'nexus3',
                            protocol: 'http',
                            nexusUrl: "$NEXUS_URL",
                            groupId: pom.groupId,
                            version: version,
                            repository: "$NEXUS_REPOSITORY",
                            credentialsId: "$NEXUS_CREDENTIALS_ID",
                            artifacts: [
                                [artifactId: artifactId, classifier: '', file: file, type: packaging]
                            ]
                        )
                    }
                }
            }
        }

        stage('Debug Info') {
            steps {
                sh 'mvn -version'
                sh 'echo $NEXUS_URL'
                sh 'echo $NEXUS_REPOSITORY'
            }
        }

        stage('Build Docker') {
            steps {
                dir('maven-app/my-app') {
                    sh 'docker build -t my-app:latest .'
                }
            }
        }

        stage('Docker Push') {
            steps {
                script {
                    withCredentials([usernamePassword(credentialsId: "$DOCKER_CREDENTIALS_ID", usernameVariable: 'DOCKER_USER', passwordVariable: 'DOCKER_PASS')]) {
                        sh 'echo $DOCKER_PASS | docker login -u $DOCKER_USER --password-stdin'
                        sh 'docker push my-app:latest'
                    }
                }
            }
        }
    }

    post {
        always {
            emailext(
                to: 'zephyr340cicd@gmail.com', // @Zephyr_340_cicd
                subject: "Jenkins Build ${currentBuild.fullDisplayName}",
                body: "Build ${currentBuild.fullDisplayName} completed with status: ${currentBuild.result}"
            )
        }
    }
}