pipeline {
    agent {
        label 'argon2'
    }

    tools {
        jdk 'OracleJDK 8'
    }

    options {
        timeout(time: 5, unit: 'MINUTES')
        timestamps()
    }

    triggers {
        githubPush()
    }

    stages {
        stage ('Checkout') {
            steps {
                script {
                    env.CI_SKIP = "false"
                    result = sh (script: "git log -1 | grep '(?s).[CI[-\\s]SKIP].*'", returnStatus: true)
                    if (result == 0) {
                        env.CI_SKIP = "true"
                        error "'[CI-SKIP]' found in git commit message. Aborting."
                    }
                }
            }
        }

        stage ('Clean') {
            steps {
                sh 'mvn -B clean'
            }
        }

        stage ('Build & Deploy') {
            steps {
                withMaven(
                    maven: 'Maven 3'
                    globalMavenSettingsConfig: 'e5b005b5-be4d-4709-8657-1981662bcbe3'
                    mavenOpts: '-Xmx4G'
                ) {
                    withCredentials([string(credentialsId: 'authme-coveralls-token', variable: 'COVERALLS_TOKEN')]) {
                        sh 'mvn -B clean package javadoc:aggregate-jar source:jar deploy coveralls:report -DrepoToken=$COVERALLS_TOKEN'
                    }
                }
            }
            post {
                success {
                    archiveArtifacts artifacts: '**/target/*.jar', excludes: '**/target/*-noshade.jar', fingerprint: true
                    step([
                        $class: 'JavadocArchiver',
                        javadocDir: 'target/apidocs',
                        keepAll: true
                    ])
                    jacoco(execPattern: '**/**.exec', classPattern: '**/classes', sourcePattern: '**/src/main/java')
                    junit 'target/surefire-reports/*.xml'
                }
            }
        }
    }

    post {
        always {
            script {
                if (env.CI_SKIP == "true") {
                    currentBuild.result = 'NOT_BUILT'
                }
            }
        }
    }
}
