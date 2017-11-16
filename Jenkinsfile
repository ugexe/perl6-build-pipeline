pipeline {
    agent none
    options {
        timestamps()
        timeout(time: 30, unit: 'MINUTES')
    }
    stages {
        stage("Build") {

        }
        stage("distribute") {
            steps {
                parallel (
                    "linux" : {
                        node('linux') {
                            post {
                                always {
                                    step([$class: "TapPublisher", testResults: "report/**/*", failIfNoResults: true, outputTapToConsole: true, showOnlyFailures: true, skipIfBuildNotOk: false, todoIsFailure: false, verbose: true])
                                }
                            }

                            sh 'mkdir -p $WORKSPACE/report/nqp'
                            sh 'mkdir -p $WORKSPACE/report/rakudo'
                            sh 'mkdir -p $WORKSPACE/report/spectest'
                            sh 'cpanm --sudo -q -n TAP::Harness::Archive'

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

                                writeFile file: ".proverc", text: "--archive \"$WORKSPACE/report/nqp\"\n--timer"
                                sh 'make test'

                                sh 'make install'
                            }
                            dir('rakudo') {
                                git url: 'https://github.com/rakudo/rakudo.git'

                                sh 'perl Configure.pl --prefix="$WORKSPACE/install"'
                                sh 'make'

                                writeFile file: ".proverc", text: "--archive \"$WORKSPACE/report/rakudo\"\n--timer"
                                sh 'make test'

                                writeFile file: ".proverc", text: "--archive \"$WORKSPACE/report/spectest\"\n--timer"
                                sh 'make spectest'

                                sh 'make install'
                            }
                        }
                    },
                    "windows" : {
                        node('windows') {
                            post {
                                always {
                                    step([$class: "TapPublisher", testResults: "report/**/*", failIfNoResults: true, outputTapToConsole: true, showOnlyFailures: true, skipIfBuildNotOk: false, todoIsFailure: false, verbose: true])
                                }
                            }

                            def shell(command) {
                                return bat(returnStdout: true, script: "C:\\Program Files (x86)\\Microsoft Visual Studio\\2017\\Community\\VC\\Auxiliary\\Build\\vcvars64.bat & \"${command}\"").trim()
                            }

                            bat 'mkdir "%WORKSPACE%\\report\\nqp"'
                            bat 'mkdir "%WORKSPACE%\\report\\rakudo"'
                            bat "mkdir \"$WORKSPACE\\report\\spectest\""
                            bat 'cpanm -q -n TAP::Harness::Archive'

                            dir('MoarVM') {
                                git url: 'https://github.com/MoarVM/MoarVM.git'

                                shell 'perl Configure.pl --prefix="%WORKSPACE%/install"'
                                shell 'nmake'

                                shell 'nmake install'
                            }
                            dir('nqp') {
                                git url: 'https://github.com/perl6/nqp.git'

                                shell "perl Configure.pl --prefix=\"$WORKSPACE/install\" --with-moar=\"$WORKSPACE/install/bin/moar\""
                                shell 'nmake'

                                writeFile file: "_proverc", text: "--archive \"$WORKSPACE/report/nqp\"\n--timer"
                                shell 'nmake test'

                                shell 'nmake install'
                            }
                            dir('rakudo') {
                                git url: 'https://github.com/rakudo/rakudo.git'

                                shell 'perl Configure.pl --prefix="%WORKSPACE%/install"'
                                shell 'nmake'

                                writeFile file: "_proverc", text: "--archive \"$WORKSPACE/report/rakudo\"\n--timer"
                                nmake test

                                writeFile file: "_proverc", text: "--archive \"$WORKSPACE/report/spectest\"\n--timer"
                                shell 'nmake spectest'

                                shell 'nmake install'
                            }
                        }
                    }
                )
            }
        }
    }
}
