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
                                sh 'prove --archive "$WORKSPACE/report/nqp" --timer -j6 -v -r -e "$WORKSPACE/install/bin/nqp-m" t/nqp t/hll t/qregex t/p5regex t/qast t/moar t/serialization t/nativecall'
                                sh 'make install'
                            }
                            dir('rakudo') {
                                git url: 'https://github.com/rakudo/rakudo.git'
                                sh 'perl Configure.pl --prefix="$WORKSPACE/install"'
                                sh 'make'
                                sh 'prove --archive "$WORKSPACE/report/rakudo" --timer -j6 -v -r -e "$WORKSPACE/install/bin/perl6-m" t/'
                                sh 'make install'
                            }
                        }
                    },
                    "windows" : {
                        node('windows') {
                            bat 'mkdir "%WORKSPACE%\\report\\nqp"'
                            bat 'mkdir "%WORKSPACE%\\report\\rakudo"'
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
                                bat '''
                                    call "C:\\Program Files (x86)\\Microsoft Visual Studio\\2017\\Community\\VC\\Auxiliary\\Build\\vcvars64.bat"
                                    prove --archive "%WORKSPACE%/report/nqp" --timer -j6 -v -r -e "%WORKSPACE%\\install\\bin\\nqp-m.bat" t/nqp t/hll t/qregex t/p5regex t/qast t/moar t/serialization t/nativecall
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
                                bat '''
                                    call "C:\\Program Files (x86)\\Microsoft Visual Studio\\2017\\Community\\VC\\Auxiliary\\Build\\vcvars64.bat"
                                    prove --archive "%WORKSPACE%/report/rakudo" --timer -j6 -v -r -e "%WORKSPACE%\\install\\bin\\perl6-m.bat" t/
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
