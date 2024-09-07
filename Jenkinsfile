pipeline {
    agent any

    environment {
        DOCKER_CREDENTIALS_ID = 'docker-credentials'
        NEXUS_CREDENTIALS_ID = 'nexus'
        SONARQUBE_CREDENTIALS_ID = 'sonar'
        SONARQUBE_URL = 'http://44.223.54.73:9000/'
        NEXUS_URL = '54.242.87.236:8081'
        NEXUS_REPOSITORY = 'circad'
        SONARQUBE_TOKEN = 'squ_0d762e5b52e55fd79db511f279eef339c771a202'
    }

    triggers {
        // Commented out since GitHub webhook is not used
        githubPush()
    }
    // sqa_e6d2389b4260b9bb4d2eaddadfaa024852daf660
    // sonar-user: squ_a792250b2379b83fbae3814afdcaabd4f3a24517
    // 22f1a9d4-13ba-36c7-97f8-b41b4835c960
    // nuget setapikey 22f1a9d4-13ba-36c7-97f8-b41b4835c960 -source http://10.1.27.202:8081/repository/{repository name}/
    tools {
        maven 'Maven 3.8.7' // Use the Maven tool configured in Jenkins
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
                script {
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