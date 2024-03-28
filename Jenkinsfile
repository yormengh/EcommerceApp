pipeline {
    agent any

    tools {
        maven "maven3.9.6"
    } 
    
    stages {
        stage ('Git clone') {
            steps {
                sh 'echo "Git cloning done.."'
                git 'https://github.com/yormengh/EcommerceApp.git'
            }
        }

        stage ('Build with Maven') {
            steps {
                // Specify the directory containing the pom.xml file
                dir('EcommerceApp') {
                    sh "mvn clean" 
                    echo "Build and cleaning done.."
                }
            }
        }

        stage ('Test with Maven') {
            steps {
                // Specify the directory containing the pom.xml file
                dir('EcommerceApp') {
                    sh "mvn test" 
                    echo "Build and Testing done.."
                }
            }
        }

        stage ('Package with Maven') {
            steps {
                // Specify the directory containing the pom.xml file
                dir('EcommerceApp') {
                    sh "mvn package" 
                    echo "Build and Package done.."
                }
            }
        }

        stage ('SonarQube Analysis') {
            environment {
                ScannerHome = tool 'sonar5.0'
            }

            steps {
                echo "Analyzing with SonarQube.."
                script {
                    dir ('EcommerceApp') {
                        def compiledClassesDir = sh(script: 'mvn help:evaluate -Dexpression=project.build.outputDirectory -q -DforceStdout', returnStdout: true).trim()

                        withSonarQubeEnv('sonarqube') {
                            sh "${ScannerHome}/bin/sonar-scanner -Dsonar.projectKey=ecommerceapp -Dsonar.java.binaries=${compiledClassesDir}"
                        }
                    }
                }
            }
        }

        stage ('Upload to Nexus') {
            steps {
                echo "Uploading to Nexus done.."
                nexusArtifactUploader artifacts: [[artifactId: 'EcommerceApp', classifier: '', file: '/var/lib/jenkins/workspace/first-pipeline-job/target/web-app.war', type: 'war']], credentialsId: 'nexus-id', groupId: 'com', nexusUrl: '18.220.20.21:8081/repository/ecommerce-webapp-snapshot/', nexusVersion: 'nexus3', protocol: 'http', repository: 'ecommerce-webapp-snapshot', version: '0.0.1-SNAPSHOT'
            }
        }

        stage ('Deploy to Tomcat') {
            steps {
                echo "Deploying to Tomcat done.."
                dir('EcommerceApp') {
                    deploy adapters: [tomcat9(credentialsId: 'tomcat-access', path: '', url: 'http://52.14.212.147:8080/')], contextPath: null, war: 'target/*.war'
                }
            }
        }
    }
}