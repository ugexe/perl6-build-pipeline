pipeline {
    agent none
    stages {
        stage("distribute") {
            steps {
                parallel (
                    "windows" : {
                        node('windows') {
                            git url: 'https://github.com/rakudo/rakudo.git'
                            git url: 'https://github.com/perl6/nqp.git'
                            git url: 'https://github.com/MoarVM/MoarVM.git'

                            bat '"C:\\Program Files (x86)\\Microsoft Visual Studio\\2017\\Community\\VC\\Auxiliary\\Build\\vcvars64.bat"'
                            bat 'dir'
                            dir('MoarVM') {
                                bat 'dir'
                                bat 'perl Configure.pl'
                                bat 'nmake'
                                bat 'nmake install'
                            }
                            dir('nqp') {
                                bat 'perl Configure.pl --prefix="%WORKSPACE%/MoarVM/install"'
                                bat 'nmake'
                                bat 'nmake test'
                                bat 'nmake install'
                            }
                            dir('rakudo') {
                                bat 'perl Configure.pl --prefix="%WORKSPACE%/nqp/install"'
                                bat 'nmake'
                                bat 'nmake test'
                                bat 'nmake install'
                            }
                        }
                    },
                    "linux" : {
                        node('linux') {
                            git url: 'https://github.com/rakudo/rakudo.git'
                            git url: 'https://github.com/perl6/nqp.git'
                            git url: 'https://github.com/MoarVM/MoarVM.git'

                            sh 'dir'

                            dir('MoarVM') {
                                sh 'dir'
                                sh 'perl Configure.pl'
                                sh 'make'
                                sh 'make install'
                            }
                            dir('nqp') {
                                sh 'perl Configure.pl --prefix="$WORKSPACE/MoarVM/install"'
                                sh 'make'
                                sh 'make test'
                                sh 'make install'
                            }
                            dir('rakudo') {
                                sh 'perl Configure.pl --prefix="$WORKSPACE/nqp/install"'
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
