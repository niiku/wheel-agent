def version = "latest"
def appName = env.APP_NAME
def dockerRegistry = env.DOCKER_REGISTRY
def currentProject

pipeline {
    agent {
        label 'maven'
    }
    stages {
        stage('checkout') {
            steps {
                checkout scm
            }
        }
        stage('clean') {
            steps {
                sh "mvn clean"
            }
        }
        stage('Determine project & version') {
            steps {
                script {
                    version = sh returnStdout: true, script: 'mvn -q -Dexec.executable=echo -Dexec.args=\'${project.version}\' --non-recursive exec:exec'
                    echo "Current project version is ${version}"
                    currentProject = ${openshift.project()}
                }
            }
        }
        stage('Run Tests') {
            steps {
                script {
                    try {
                        sh "mvn test"
                    } catch (err) {
                        throw err
                    } finally {
                        //     junit '**/buildTEST-*.xml'
                    }
                }
            }
        }

        stage('Packaging') {
            steps {
                sh "mvn package"
            }
        }

        stage('Create Image Builder') {
            when {
                expression {
                    openshift.withCluster() {
                        openshift.withProject() {
                            return !openshift.selector("bc", appName).exists()
                        }
                    }
                }
            }
            steps {
                script {
                    openshift.withCluster() {
                        openshift.withProject(devProject) {
                            openshift.newBuild("--name=${appName}", "--strategy docker", "--binary=true", "--docker-image openjdk:8-jre-alpine")
                        }
                    }
                }
            }
        }

        stage('Build Image') {
            steps {
                script {
                    openshift.withCluster() {
                        openshift.withProject() {
                            openshift.selector("bc", appName).startBuild("--from-dir=.", "--wait=true", "--follow")
                            openshift.tag("${currentProject}/${appName}:latest", "${currentProject}/${appName}:${version}")
                        }
                    }
                }
            }
        }
    }
}