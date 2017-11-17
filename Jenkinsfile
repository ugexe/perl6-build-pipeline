pipeline {
    agent none
    environment {
        ALLOW_PASSING_TODOS = 1
    }
    options {
        timestamps()
        timeout(time: 30, unit: 'MINUTES')
    }
    stages {
        stage("all") {
            parallel {
                stage("all on linux") {
                    agent {
                        label "linux"
                    }
                    post {
                        always {
                            junit "report/**/*.xml"
                        }
                    }
                    steps {
                        sh 'mkdir -p $WORKSPACE/report/nqp'
                        sh 'mkdir -p $WORKSPACE/report/rakudo'
                        sh 'mkdir -p $WORKSPACE/report/spectest'
                        sh 'cpanm --sudo -q -n TAP::Harness::Archive TAP::Formatter::JUnitREGRU'

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

                            withEnv(['PERL_TEST_HARNESS_DUMP_TAP' = "$WORKSPACE/report/nqp"']) {
                                writeFile file: ".proverc", text: "--formatter TAP::Formatter::JUnitREGRU\n--timer"
                                sh 'make test'
                            }

                            sh 'make install'
                        }
                        dir('rakudo') {
                            git url: 'https://github.com/rakudo/rakudo.git'

                            sh 'perl Configure.pl --prefix="$WORKSPACE/install"'
                            sh 'make'

                            withEnv(['PERL_TEST_HARNESS_DUMP_TAP' = "$WORKSPACE/report/rakudo"']) {
                                writeFile file: ".proverc", text: "--formatter TAP::Formatter::JUnitREGRU\n--timer"
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
