pipeline {
    agent none
    options {
        timestamps()
        timeout(time: 30, unit: 'MINUTES')
    }
    stages {
        stage("Distribute") {
            parallel {
                stage("Build on Linux") {
                    agent {
                        label "linux"
                    }
                    environment {
                       ALLOW_PASSING_TODOS=1
                       TEST_DUMP_DIR="report"
                    }
                    post {
                        always {
                            junit "**/$TEMP_DUMP_DIR/**/*.xml"
                        }
                    }
                    steps {
                        dir('MoarVM') {
                            git url: 'https://github.com/MoarVM/MoarVM.git'

                            sh 'perl Configure.pl --prefix="$WORKSPACE/install"'
                            sh 'make'

                            sh 'make install'
                        }

                        dir('nqp') {
                            git url: 'https://github.com/perl6/nqp.git'

                            sh 'perl Configure.pl --prefix="$WORKSPACE/install" --with-moar="$WORKSPACE/install/bin/moar"'
                            sh 'make'

                            withEnv(['PERL_TEST_HARNESS_DUMP_TAP=$TEST_DUMP_DIR/nqp', 'ALLOW_PASSING_TODOS=1']) {
                                sh 'mkdir -p "$PERL_TEST_HARNESS_DUMP_TAP"'
                                writeFile file: ".proverc", text: "--formatter TAP::Formatter::JUnitREGRU"
                                sh 'make test'
                            }

                            sh 'make install'
                        }

                        dir('rakudo') {
                            git url: 'https://github.com/rakudo/rakudo.git'

                            sh 'perl Configure.pl --prefix="$WORKSPACE/install"'
                            sh 'make'

                            withEnv(['PERL_TEST_HARNESS_DUMP_TAP=$TEST_DUMP_DIR/rakudo', 'ALLOW_PASSING_TODOS=1']) {
                                sh 'mkdir -p "$PERL_TEST_HARNESS_DUMP_TAP"'
                                writeFile file: ".proverc", text: "--formatter TAP::Formatter::JUnitREGRU"
                                sh 'make test'
                            }

                            sh 'make install'
                        }

                        dir('rakudo') {
                            withEnv(['PERL_TEST_HARNESS_DUMP_TAP=$TEST_DUMP_DIR/spectest', 'ALLOW_PASSING_TODOS=1']) {
                                sh 'mkdir -p "$PERL_TEST_HARNESS_DUMP_TAP"'
                                writeFile file: ".proverc", text: "--formatter TAP::Formatter::JUnitREGRU"
                                sh 'make spectest'
                            }
                        }
                    }
                }
            }
        }
    }
}
