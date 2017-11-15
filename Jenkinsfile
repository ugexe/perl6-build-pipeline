pipeline {
    agent none
    stages {
        stage("distribute") {
            steps {
                parallel (
                    "windows" : {
                        node('windows') {
                            bat 'mkdir install'

                            dir('MoarVM') {
                                git url: 'https://github.com/MoarVM/MoarVM.git'
                                bat '''
                                    call "C:\\Program Files (x86)\\Microsoft Visual Studio\\2017\\Community\\Common7\\Tools\\VsMSBuildCmd.bat"
                                    call "C:\\Program Files (x86)\\Microsoft Visual Studio\\2017\\Community\\VC\\Auxiliary\\Build\\vcvars64.bat"
                                    call perl Configure.pl --prefix="%WORKSPACE%/install"
                                    nmake
                                    nmake install
                                '''
                            }
                            dir('nqp') {
                                git url: 'https://github.com/perl6/nqp.git'
                                bat '''
                                    call "C:\\Program Files (x86)\\Microsoft Visual Studio\\2017\\Community\\Common7\\Tools\\VsMSBuildCmd.bat"
                                    call "C:\\Program Files (x86)\\Microsoft Visual Studio\\2017\\Community\\VC\\Auxiliary\\Build\\vcvars64.bat"
                                    call perl Configure.pl --prefix="%WORKSPACE%/install"
                                    call nmake
                                    call nmake test
                                    call nmake install
                                '''
                            }
                            dir('rakudo') {
                                git url: 'https://github.com/rakudo/rakudo.git'
                                bat '''
                                    call "C:\\Program Files (x86)\\Microsoft Visual Studio\\2017\\Community\\Common7\\Tools\\VsMSBuildCmd.bat"
                                    call "C:\\Program Files (x86)\\Microsoft Visual Studio\\2017\\Community\\VC\\Auxiliary\\Build\\vcvars64.bat"
                                    call perl Configure.pl --prefix="%WORKSPACE%/install"
                                    call nmake
                                    call nmake test
                                    call nmake install
                                '''
                            }
                        }
                    },
                    "linux" : {
                        node('linux') {
                            sh 'mkdir install'

                            dir('MoarVM') {
                                git url: 'https://github.com/MoarVM/MoarVM.git'
                                sh 'perl Configure.pl --prefix="$WORKSPACE/install"'
                                sh 'make'
                                sh 'make install'
                            }
                            dir('nqp') {
                                git url: 'https://github.com/perl6/nqp.git'
                                sh 'perl Configure.pl --prefix="$WORKSPACE/install" --with-moar="$WORKSPACE/install"'
                                sh 'make'
                                sh 'make test'
                                sh 'make install'
                            }
                            dir('rakudo') {
                                git url: 'https://github.com/rakudo/rakudo.git'
                                sh 'perl Configure.pl --prefix="$WORKSPACE/install"'
                                sh 'make'
                                sh 'make test'
                                sh 'make install'
                            }
                        }
                    }
                )
            }
        }
    }    
}
