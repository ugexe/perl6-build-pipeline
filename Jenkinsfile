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
                    }
                    post {
                        always {
                            sh 'ls report -LR'
                            junit "**/report/**/*.xml"
                        }
                    }
                    steps {
                        sh 'mkdir -p $WORKSPACE/report/rakudo'
                        sh 'mkdir -p $WORKSPACE/report/spectest'

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

                            withEnv(['PERL_TEST_HARNESS_DUMP_TAP=report/nqp', 'ALLOW_PASSING_TODOS=1']) {
                                sh 'mkdir -p "$PERL_TEST_HARNESS_DUMP_TAP"'
                                writeFile file: ".foo", text: "--archive \"$PERL_TEST_HARNESS_DUMP_TAP\"\n--formatter TAP::Formatter::JUnitREGRU"
                                sh 'cat .foo'

                                writeFile file: ".proverc", text: "--formatter TAP::Formatter::JUnitREGRU"
                                sh 'cat .proverc'
                                sh 'make test'
                            }

                            sh 'make install'
                        }
                    }
                }
            }
        }
    }
}
