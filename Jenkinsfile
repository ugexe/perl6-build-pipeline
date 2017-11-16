pipeline {
    agent none
    options {
        timestamps()
        timeout(time: 20, unit: 'MINUTES')
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
                                writeFile file: ".proverc", text: "--archive \"$WORKSPACE/report/rakudo\"\n--timer"
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
                            bat 'mkdir "%WORKSPACE%\\report\\nqp"'
                            bat 'mkdir "%WORKSPACE%\\report\\rakudo"'
                            bat 'mkdir "%WORKSPACE%\\report\\spectest"'
                            bat 'cpanm -q -n TAP::Harness::Archive'

                            dir('MoarVM') {
                                git url: 'https://github.com/MoarVM/MoarVM.git'
                                bat '''
                                    call "C:\\Program Files (x86)\\Microsoft Visual Studio\\2017\\Community\\VC\\Auxiliary\\Build\\vcvars64.bat"
                                    perl Configure.pl --prefix="%WORKSPACE%/install"
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
                                    perl Configure.pl --prefix="%WORKSPACE%/install" --with-moar="%WORKSPACE%/install/bin/moar"
                                '''
                                bat '''
                                    call "C:\\Program Files (x86)\\Microsoft Visual Studio\\2017\\Community\\VC\\Auxiliary\\Build\\vcvars64.bat"
                                    nmake
                                '''
                                writeFile file: "_proverc", text: "--archive \"$WORKSPACE/repor/nqp\"\n--timer"
                                bat '''
                                    call "C:\\Program Files (x86)\\Microsoft Visual Studio\\2017\\Community\\VC\\Auxiliary\\Build\\vcvars64.bat"
                                    nmake test
                                '''
                                bat '''
                                    call "C:\\Program Files (x86)\\Microsoft Visual Studio\\2017\\Community\\VC\\Auxiliary\\Build\\vcvars64.bat"
                                    nmake install
                                '''
                            }
                            dir('rakudo') {
                                git url: 'https://github.com/rakudo/rakudo.git'

                                bat '''
                                    call "C:\\Program Files (x86)\\Microsoft Visual Studio\\2017\\Community\\VC\\Auxiliary\\Build\\vcvars64.bat"
                                    perl Configure.pl --prefix="%WORKSPACE%/install"
                                '''
                                bat '''
                                    call "C:\\Program Files (x86)\\Microsoft Visual Studio\\2017\\Community\\VC\\Auxiliary\\Build\\vcvars64.bat"
                                    nmake
                                '''
                                writeFile file: "_proverc", text: "--archive \"$WORKSPACE/report/rakudo\"\n--timer"
                                bat '''
                                    call "C:\\Program Files (x86)\\Microsoft Visual Studio\\2017\\Community\\VC\\Auxiliary\\Build\\vcvars64.bat"
                                    nmake test
                                '''
                                writeFile file: "_proverc", text: "--archive \"$WORKSPACE/report/spectest\"\n--timer"
                                bat '''
                                    call "C:\\Program Files (x86)\\Microsoft Visual Studio\\2017\\Community\\VC\\Auxiliary\\Build\\vcvars64.bat"
                                    nmake spectest
                                '''
                                bat '''
                                    call "C:\\Program Files (x86)\\Microsoft Visual Studio\\2017\\Community\\VC\\Auxiliary\\Build\\vcvars64.bat"
                                    nmake install
                                '''
                            }
                        }
                    }
                )
            }
        }
    }
    post {
        always {
            step([$class: "TapPublisher", testResults: "$WORKSPACE/report/**/*.t", failIfNoResults: true, outputTapToConsole: true, showOnlyFailures: true, skipIfBuildNotOk: true])
        }
    }
}
