pipeline {
    agent any

        stages {
            stage('Source') {
                steps {
                    git url: 'https://github.com/Bangaruyogi77/Mysql-spring-demo.git'
                }
            }
            stage('Build') {
                steps {
                    script {
                        def mvnHome = tool 'M3'
                        bat "${mvnHome}\\bin\\mvn -B verify"
                    }
                }
            }
            stage('SonarQube Analysis') {
                steps {
                    script {
                        def mvnHome = tool 'M3'
                        withSonarQubeEnv() {
                            bat "${mvnHome}/bin/mvn clean verify sonar:sonar -Dsonar.projectKey=mysql-sonar"
                        }
                    }
                }
            }
    
            stage('Packaging') {
                steps {
                    step([$class: 'ArtifactArchiver', artifacts: '*/target/.jar', fingerprint: true])
                }
            }
            
            stage ("Artifactory Publish"){
                steps{
                    script{
                            def server = Artifactory.server 'artifactory'
                            def rtMaven = Artifactory.newMavenBuild()
                            //rtMaven.resolver server: server, releaseRepo: 'jenkins-devops', snapshotRepo: 'jenkins-devops-snapshot'
                            rtMaven.deployer server: server, releaseRepo: 'mysql', snapshotRepo: 'mysql-snapshot'
                            rtMaven.tool = 'M3'
                            
                            def buildInfo = rtMaven.run pom: '$workspace/pom.xml', goals: 'clean install'
                            rtMaven.deployer.deployArtifacts = true
                            rtMaven.deployer.deployArtifacts buildInfo

                            server.publishBuildInfo buildInfo
                    }
                }
        }
        }
}
