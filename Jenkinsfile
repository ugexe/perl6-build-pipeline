pipeline {
    agent none
    options {
        timestamps()
        timeout(time: 60, unit: 'MINUTES')
    }
    stages {
        stage("Build") {
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

                            withEnv(['PERL_TEST_HARNESS_DUMP_TAP=$TEST_DUMP_DIR/nqp']) {
                                timeout(time: 5, unit: 'MINUTES') {
                                    sh(returnStatus: true, script: '''
                                        mkdir -p "$PERL_TEST_HARNESS_DUMP_TAP"
                                        prove --formatter TAP::Formatter::JUnitREGRU --timer -r -j4 -e ./nqp t/nqp t/moar t/hll t/qast t/qregex t/serialization
                                    ''')
                                }
                            }

                            sh 'make install'
                        }

                        dir('rakudo') {
                            git url: 'https://github.com/rakudo/rakudo.git'

                            sh 'perl Configure.pl --prefix="$INSTALL_DIR"'
                            sh 'make'

                            withEnv(['PERL_TEST_HARNESS_DUMP_TAP=$TEST_DUMP_DIR/rakudo']) {
                                timeout(time: 20, unit: 'MINUTES') {
                                    sh(returnStatus: true, script: '''
                                        mkdir -p "$PERL_TEST_HARNESS_DUMP_TAP"
                                        prove --formatter TAP::Formatter::JUnitREGRU --timer -r -j4 -e './perl6 -Ilib' t/
                                    ''')
                                }
                            }

                            sh 'make install'
                        }

                        dir('spectest') {
                            git url: 'https://github.com/perl6/roast.git'

                            withEnv(['PATH+=$INSTALL_DIR/bin','PERL_TEST_HARNESS_DUMP_TAP=$TEST_DUMP_DIR/spectest']) {
                                sh 'printenv'
                                timeout(time: 30, unit: 'MINUTES') {
                                    sh(returnStatus: true, script: '''
                                        mkdir -p "$PERL_TEST_HARNESS_DUMP_TAP"
                                        perl fudgeall rakudo.moar **/*.t > test-list-spaces.txt
                                        perl -p -e \'s/\\s+/\\n/g\' test-list-spaces.txt > test-list.txt
                                        prove --formatter TAP::Formatter::JUnitREGRU --timer -r -j4 -e \"$INSTALL_DIR/bin/perl6 -I "$INSTALL_DIR/lib" -Ipackages \" - < test-list.txt
                                    ''')
                                }
                            }
                        }
                    }
                }

                stage("Build on Windows") {
                    agent {
                        label "windows"
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
                        bat 'cpanm -v -n TAP::Formatter::JUnitREGRU'

                        dir('MoarVM') {
                            git url: 'https://github.com/MoarVM/MoarVM.git'

                            bat '''
                                call "C:\\Program Files (x86)\\Microsoft Visual Studio\\2017\\Community\\VC\\Auxiliary\\Build\\vcvars64.bat"
                                perl Configure.pl --prefix="%INSTALL_DIR%"
                            '''
                            bat '''
                                call "C:\\Program Files (x86)\\Microsoft Visual Studio\\2017\\Community\\VC\\Auxiliary\\Build\\vcvars64.bat"
                                nmake
                            '''

                            bat '''
                                call "C:\\Program Files (x86)\\Microsoft Visual Studio\\2017\\Community\\VC\\Auxiliary\\Build\\vcvars64.bat"
                                nmake install
                            '''
                        }

                        dir('nqp') {
                            git url: 'https://github.com/perl6/nqp.git'

                            bat '''
                                call "C:\\Program Files (x86)\\Microsoft Visual Studio\\2017\\Community\\VC\\Auxiliary\\Build\\vcvars64.bat"
                                perl Configure.pl --prefix="%INSTALL_DIR%" --with-moar="%INSTALL_DIR%\\bin\\moar"
                            '''
                            bat '''
                                call "C:\\Program Files (x86)\\Microsoft Visual Studio\\2017\\Community\\VC\\Auxiliary\\Build\\vcvars64.bat"
                                nmake
                            '''

                            withEnv(['PERL_TEST_HARNESS_DUMP_TAP=$TEST_DUMP_DIR/nqp']) {
                                timeout(time: 5, unit: 'MINUTES') {
                                    bat(returnStatus: true, script: '''
                                        mkdir "$PERL_TEST_HARNESS_DUMP_TAP"
                                        call "C:\\Program Files (x86)\\Microsoft Visual Studio\\2017\\Community\\VC\\Auxiliary\\Build\\vcvars64.bat"
                                        prove --formatter TAP::Formatter::JUnitREGRU --timer -r -j4 -e ".\\nqp" t/nqp t/moar t/hll t/qast t/qregex t/serialization
                                    ''')
                                }
                            }

                            bat '''
                                call "C:\\Program Files (x86)\\Microsoft Visual Studio\\2017\\Community\\VC\\Auxiliary\\Build\\vcvars64.bat"
                                nmake install
                            '''
                        }

                        dir('rakudo') {
                            git url: 'https://github.com/rakudo/rakudo.git'

                            bat '''
                                call "C:\\Program Files (x86)\\Microsoft Visual Studio\\2017\\Community\\VC\\Auxiliary\\Build\\vcvars64.bat"
                                perl Configure.pl --prefix="%INSTALL_DIR%"
                            '''
                            bat '''
                                call "C:\\Program Files (x86)\\Microsoft Visual Studio\\2017\\Community\\VC\\Auxiliary\\Build\\vcvars64.bat"
                                nmake
                            '''

                            withEnv(['PERL_TEST_HARNESS_DUMP_TAP=$TEST_DUMP_DIR/rakudo']) {
                                timeout(time: 10, unit: 'MINUTES') {
                                    bat(returnStatus: true, script: '''
                                        mkdir "%PERL_TEST_HARNESS_DUMP_TAP%"
                                        call "C:\\Program Files (x86)\\Microsoft Visual Studio\\2017\\Community\\VC\\Auxiliary\\Build\\vcvars64.bat"
                                        prove --formatter TAP::Formatter::JUnitREGRU --timer -r -j4 -e ".\\perl6 -Ilib" t/
                                        nmake test
                                    ''')
                                }
                            }


                            bat '''
                                call "C:\\Program Files (x86)\\Microsoft Visual Studio\\2017\\Community\\VC\\Auxiliary\\Build\\vcvars64.bat"
                                nmake install
                            '''
                        }

                        dir('spectest') {
                            git url: 'https://github.com/perl6/roast.git'

                            withEnv(['PATH+=$INSTALL_DIR/bin','PERL_TEST_HARNESS_DUMP_TAP=$TEST_DUMP_DIR/spectest', 'ALLOW_PASSING_TODOS=1']) {
                                timeout(time: 30, unit: 'MINUTES') {
                                    bat(returnStatus: true, script: '''
                                        mkdir "%PERL_TEST_HARNESS_DUMP_TAP%"
                                        perl fudgeall rakudo.moar **/*.t > test-list-spaces.txt
                                        perl -p -e "s/\\s+/\\n/g" test-list-spaces.txt > test-list.txt
                                        prove --formatter TAP::Formatter::JUnitREGRU" --timer -r -j4 -e "\"%INSTALL_DIR%\\bin\\perl6-m.bat\" -I \"%INSTALL_DIR%/lib\" -Ipackages " - < test-list.txt
                                    ''')
                                }
                            }
                        }
                    }
                }
            }
        }
    }
}
