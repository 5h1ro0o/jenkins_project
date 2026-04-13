/**
 * Jenkinsfile – Pipeline CI complète
 * Projet : Boutique en ligne – ICDE848
 * Adapté Windows (bat au lieu de sh)
 */

pipeline {

    agent any

    tools {
        maven 'Maven3'
        jdk   'JDK17'
    }

    parameters {
        string(
            name:         'BRANCH',
            defaultValue: 'main',
            description:  'Branche Git à builder'
        )
        choice(
            name:    'ENVIRONMENT',
            choices: ['dev', 'staging', 'prod'],
            description: 'Environnement de déploiement cible'
        )
        booleanParam(
            name:         'SKIP_TESTS',
            defaultValue: false,
            description:  'Ignorer les tests (urgence uniquement !)'
        )
    }

    stages {

        // ── Stage 1 : Checkout ────────────────────────
        stage('Checkout') {
            steps {
                checkout scm
                echo "Branch  : ${env.GIT_BRANCH}"
                echo "Commit  : ${env.GIT_COMMIT}"
            }
        }

        // ── Stage 2 : Build ───────────────────────────
        stage('Build') {
            steps {
                bat 'mvn clean compile -B'
            }
        }

        // ── Stage 3 : Tests unitaires ─────────────────
        stage('Tests unitaires') {
            when {
                not { expression { return params.SKIP_TESTS } }
            }
            steps {
                bat 'mvn test -B'
            }
            post {
                always {
                    junit '**/target/surefire-reports/*.xml'
                }
                failure {
                    echo 'Tests unitaires en ECHEC — vérifier les logs ci-dessus'
                }
            }
        }

        // ── Stage 4 : Tests intégration ───────────────
        stage('Tests integration') {
            when {
                not { expression { return params.SKIP_TESTS } }
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

        // ── Stage 5 : Couverture JaCoCo ───────────────
        stage('Couverture JaCoCo') {
            steps {
                bat 'mvn jacoco:report -B'
            }
            post {
                always {
                    jacoco(
                        execPattern:        '**/target/jacoco.exec',
                        classPattern:       '**/target/classes',
                        sourcePattern:      '**/src/main/java',
                        minimumLineCoverage: '70'
                    )
                }
            }
        }

        // ── Stage 6 : Qualité ─────────────────────────
        stage('Qualite') {
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
                        qualityGates: [[
                            threshold: 10,
                            type: 'TOTAL',
                            unstable: true
                        ]]
                    )
                }
            }
        }

        // ── Stage 7 : Archive ─────────────────────────
        stage('Archive') {
            steps {
                archiveArtifacts(
                    artifacts:         '**/target/*.jar',
                    fingerprint:       true,
                    allowEmptyArchive: false
                )
                echo "Artefact archive avec succes"
            }
        }

    } // fin stages

    // ── POST ──────────────────────────────────────────
    post {

        always {
            echo "Pipeline terminee -- statut : ${currentBuild.currentResult}"
        }

        failure {
            emailext(
                subject: "FAILED: ${env.JOB_NAME} #${env.BUILD_NUMBER}",
                body: """
Le build a echoue.

Projet  : ${env.JOB_NAME}
Build   : #${env.BUILD_NUMBER}
Branche : ${env.GIT_BRANCH}
URL     : ${env.BUILD_URL}

Consulter les logs : ${env.BUILD_URL}console
                """,
                to:        'ton.adresse@gmail.com',
                attachLog: true
            )
        }

        fixed {
            emailext(
                subject: "FIXED: ${env.JOB_NAME} #${env.BUILD_NUMBER}",
                body:    "Le build est de nouveau stable : ${env.BUILD_URL}",
                to:      'ton.adresse@gmail.com'
            )
        }

    } // fin post

} // fin pipeline