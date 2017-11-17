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
                                step([$class: "TapPublisher", testResults: "$WORKSPACE/report/nqp/*", failIfNoResults: true, outputTapToConsole: true, showOnlyFailures: true, skipIfBuildNotOk: false, todoIsFailure: false, verbose: true])

                                sh 'make install'
                            }
                            dir('rakudo') {
                                git url: 'https://github.com/rakudo/rakudo.git'

                                sh 'perl Configure.pl --prefix="$WORKSPACE/install"'
                                sh 'make'

                                writeFile file: ".proverc", text: "--archive \"$WORKSPACE/report/rakudo\"\n--timer"
                                sh 'make test'
                                step([$class: "TapPublisher", testResults: "$WORKSPACE/report/rakudo/*", failIfNoResults: true, outputTapToConsole: true, showOnlyFailures: true, skipIfBuildNotOk: false, todoIsFailure: false, verbose: true])

                                writeFile file: ".proverc", text: "--archive \"$WORKSPACE/report/spectest\"\n--timer"
                                sh 'make spectest'

                                sh 'make install'
                            }
                        }
                    }
                )
            }
        }
    }
}
