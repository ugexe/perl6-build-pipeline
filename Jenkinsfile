pipeline {
    agent none
    options {
        timestamps()
        timeout(time: 60, unit: 'MINUTES')
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
                       INSTALL_DIR="$WORKSPACE/install"
                    }
                    post {
                        always {
                            junit "**/$TEMP_DUMP_DIR/**/*.xml"
                        }
                    }
                    steps {
                        dir('MoarVM') {
                            git url: 'https://github.com/MoarVM/MoarVM.git'

                            sh 'perl Configure.pl --prefix="$INSTALL_DIR"'
                            sh 'make'

                            sh 'make install'
                        }

                        dir('nqp') {
                            git url: 'https://github.com/perl6/nqp.git'

                            sh 'perl Configure.pl --prefix="$INSTALL_DIR" --with-moar="$INSTALL_DIR/bin/moar"'
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

                            sh 'perl Configure.pl --prefix="$INSTALL_DIR"'
                            sh 'make'

                            withEnv(['PERL_TEST_HARNESS_DUMP_TAP=$TEST_DUMP_DIR/rakudo', 'ALLOW_PASSING_TODOS=1']) {
                                sh 'mkdir -p "$PERL_TEST_HARNESS_DUMP_TAP"'
                                writeFile file: ".proverc", text: "--formatter TAP::Formatter::JUnitREGRU"
                                sh 'make test'
                            }

                            sh 'make install'
                        }

                        dir('spectest') {
                            git url: 'https://github.com/perl6/roast.git'

                            withEnv(['PATH+=$INSTALL_DIR/bin','PERL_TEST_HARNESS_DUMP_TAP=$TEST_DUMP_DIR/spectest', 'ALLOW_PASSING_TODOS=1']) {
                                sh 'mkdir -p "$PERL_TEST_HARNESS_DUMP_TAP"'

                                sh '''
                                    perl fudgeall **/*.t > test-list-spaces.txt
                                    perl -p -e 's/\s+/\n/g' test-list-spaces.txt > test-list.txt
                                    prove -e 'perl6' - < test-list.txt
                                '''
                                writeFile file: ".proverc", text: "--formatter TAP::Formatter::JUnitREGRU"
                                sh 'prove -e "perl6 -I \"$INSTALL_DIR/lib\""'
                                sh 'make spectest'
                            }
                        }
                    }
                }
            }
        }
    }
}
