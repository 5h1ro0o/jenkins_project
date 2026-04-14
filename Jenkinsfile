pipeline {

    agent any

    tools {
        maven 'Maven3'
        jdk   'JDK17'
    }

    stages {

        stage('Checkout') {
            steps {
                checkout scm
                echo "Branche : ${env.BRANCH_NAME}"
                echo "Commit  : ${env.GIT_COMMIT}"
            }
        }

        stage('Build') {
            steps {
                bat 'mvn clean compile -B'
            }
        }

        stage('Tests unitaires') {
            steps {
                bat 'mvn test -B'
            }
            post {
                always {
                    junit '**/target/surefire-reports/*.xml'
                }
            }
        }

        stage('Tests integration') {
            when {
                anyOf {
                    branch 'main'
                    branch 'develop'
                }
            }
            steps {
                bat 'mvn verify -Dsurefire.skip=true -B'
            }
            post {
                always {
                    junit '**/target/failsafe-reports/*.xml'
                }
            }
        }

        stage('Couverture JaCoCo') {
            when {
                anyOf {
                    branch 'main'
                    branch 'develop'
                }
            }
            steps {
                bat 'mvn jacoco:report -B'
            }
            post {
                always {
                    jacoco(
                        execPattern:         '**/target/jacoco.exec',
                        classPattern:        '**/target/classes',
                        sourcePattern:       '**/src/main/java',
                        minimumLineCoverage: '70'
                    )
                }
            }
        }

        stage('Qualite') {
            when {
                anyOf {
                    branch 'main'
                    branch 'develop'
                }
            }
            steps {
                bat 'mvn checkstyle:checkstyle pmd:pmd pmd:cpd spotbugs:spotbugs -B'
            }
            post {
                always {
                    recordIssues(
                        enabledForFailure: true,
                        tools: [
                            checkStyle(pattern: '**/checkstyle-result.xml'),
                            pmdParser(pattern:  '**/pmd.xml'),
                            cpd(pattern:        '**/cpd.xml'),
                            spotBugs(pattern:   '**/spotbugsXml.xml')
                        ],
                        qualityGates: [[threshold: 10, type: 'TOTAL', unstable: true]]
                    )
                }
            }
        }

        stage('Archive') {
            when {
                branch 'main'
            }
            steps {
                archiveArtifacts(
                    artifacts:         '**/target/*.jar',
                    fingerprint:       true,
                    allowEmptyArchive: false
                )
                echo 'Artefact archive sur main'
            }
        }

    } // fin stages

    post {

        always {
            echo "Pipeline terminee [${env.BRANCH_NAME}] -- ${currentBuild.currentResult}"
        }

        failure {
            emailext(
                subject: "FAILED [${env.BRANCH_NAME}]: ${env.JOB_NAME} #${env.BUILD_NUMBER}",
                body: """
Build echoue sur la branche ${env.BRANCH_NAME}.
URL : ${env.BUILD_URL}console
                """,
                to:        'ton.adresse@gmail.com',
                attachLog: true
            )
        }

        fixed {
            emailext(
                subject: "FIXED [${env.BRANCH_NAME}]: ${env.JOB_NAME} #${env.BUILD_NUMBER}",
                body:    "Build stable sur ${env.BRANCH_NAME} : ${env.BUILD_URL}",
                to:      'ton.adresse@gmail.com'
            )
        }

    } // fin post

} // fin pipeline
