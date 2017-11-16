#!/bin/groovy

def shell(Map params = [:]) {
    String script = params.script
    Boolean returnStatus = params.get('returnStatus', false)
    Boolean returnStdout = params.get('returnStdout', false)
    String encoding = params.get('encoding', null)

    if (isUnix()) {
        return steps.sh(script: script,
                  returnStatus: returnStatus,
                  returnStdout: returnStdout,
                      encoding: encoding)
    } else {
        return steps.powershell(script: script,
                  returnStatus: returnStatus,
                  returnStdout: returnStdout,
                      encoding: encoding)    
    }
}
def shell(String script) {
    return shell(script: script)
}

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

                            shell('mkdir -p $WORKSPACE/report/nqp')
                            shell('mkdir -p $WORKSPACE/report/rakudo')
                            shell('mkdir -p $WORKSPACE/report/spectest')
                            shell('cpanm --sudo -q -n TAP::Harness::Archive')

                            dir('MoarVM') {
                                git url: 'https://github.com/MoarVM/MoarVM.git'

                                shell('perl Configure.pl --prefix="$WORKSPACE/install"')
                                shell('make')

                                shell('make install')
                            }
                            dir('nqp') {
                                git url: 'https://github.com/perl6/nqp.git'

                                shell('perl Configure.pl --prefix="$WORKSPACE/install" --with-moar="$WORKSPACE/install/bin/moar"')
                                shell('make')

                                writeFile file: ".proverc", text: "--archive \"$WORKSPACE/report/nqp\"\n--timer"
                                shell('make test')

                                shell('make install')
                            }
                            dir('rakudo') {
                                git url: 'https://github.com/rakudo/rakudo.git'

                                shell('perl Configure.pl --prefix="$WORKSPACE/install"')
                                shell('make')

                                writeFile file: ".proverc", text: "--archive \"$WORKSPACE/report/rakudo\"\n--timer"
                                shell('make test')

                                writeFile file: ".proverc", text: "--archive \"$WORKSPACE/report/spectest\"\n--timer"
                                shell('make spectest')

                                shell('make install')
                            }
                        }
                    }
                )
            }
        }
    }
}
