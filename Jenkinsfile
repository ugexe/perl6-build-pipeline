pipeline {
    agent none
    options {
        timestamps()
        timeout(time: 30, unit: 'MINUTES')
    }
    stages {
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
                            def foo = { String command ->
                                if (isUnix()) {
                                    return sh(returnStdout: true, script: """
                                        sh \"${command}"
                                    """)
                                } else {
                                    return powershell(returnStdout: true, script: """
                                        call \"C:\\Program Files (x86)\\Microsoft Visual Studio\\2017\\Community\\VC\\Auxiliary\\Build\\vcvars64.bat\"
                                        Invoke-Expression \"${command}\"
                                    """)
                                }
                            }

                            shell 'mkdir -p $WORKSPACE/report/nqp'
                            shell 'mkdir -p $WORKSPACE/report/rakudo'
                            shell 'mkdir -p $WORKSPACE/report/spectest'
                            shell 'cpanm --sudo -q -n TAP::Harness::Archive'

                            dir('MoarVM') {
                                git url: 'https://github.com/MoarVM/MoarVM.git'

                                shell 'perl Configure.pl --prefix="$WORKSPACE/install"'
                                shell 'make'

                                shell 'make install'
                            }
                            dir('nqp') {
                                git url: 'https://github.com/perl6/nqp.git'

                                shell 'perl Configure.pl --prefix="$WORKSPACE/install" --with-moar="$WORKSPACE/install/bin/moar"'
                                shell 'make'

                                writeFile file: ".proverc", text: "--archive \"$WORKSPACE/report/nqp\"\n--timer"
                                shell 'make test'

                                shell 'make install'
                            }
                            dir('rakudo') {
                                git url: 'https://github.com/rakudo/rakudo.git'

                                shell 'perl Configure.pl --prefix="$WORKSPACE/install"'
                                shell 'make'

                                writeFile file: ".proverc", text: "--archive \"$WORKSPACE/report/rakudo\"\n--timer"
                                shell 'make test'

                                writeFile file: ".proverc", text: "--archive \"$WORKSPACE/report/spectest\"\n--timer"
                                shell 'make spectest'

                                shell 'make install'
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
                            def foo = { String command ->
                                if (isUnix()) {
                                    return sh(returnStdout: true, script: """
                                        sh \"${command}"
                                    """)
                                } else {
                                    return powershell(returnStdout: true, script: """
                                        call \"C:\\Program Files (x86)\\Microsoft Visual Studio\\2017\\Community\\VC\\Auxiliary\\Build\\vcvars64.bat\"
                                        Invoke-Expression \"${command}\"
                                    """)
                                }
                            }

                            shell 'mkdir "%WORKSPACE%\\report\\nqp"'
                            shell 'mkdir "%WORKSPACE%\\report\\rakudo"'
                            shell "mkdir \"$WORKSPACE\\report\\spectest\""
                            shell 'cpanm -q -n TAP::Harness::Archive'

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
