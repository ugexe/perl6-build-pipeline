pipeline {
    agent none
    stages {
        stage("distribute") {
            steps {
                parallel (
                    "windows" : {
                        node('windows') {
                            stage('Clone sources') {
                                git url: 'git@github.com:rakudo/rakudo.git'
                                git url: 'git@github.com:perl6/nqp.git'
                                git url: 'git@github.com:MoarVM/MoarVM.git'
                            }

                            bat 'call "C:\\Program Files (x86)\\Microsoft Visual Studio\\2017\\Community\\VC\\Auxiliary\\Build\\vcvars64.bat"'

                            dir('MoarVM') {
                                bat 'call perl Configure.pl'
                                bat 'call nmake'
                                bat 'call nmake install'
                            }
                            dir('nqp') {
                                bat 'call perl Configure.pl --prefix="%WORKSPACE%/MoarVM/install"'
                                bat 'call nmake'
                                bat 'call nmake install'
                            }
                            dir('rakudo') {
                                bat 'call perl Configure.pl --prefix="%WORKSPACE%/nqp/install"'
                                bat 'call nmake'
                                bat 'call nmake install'
                            }
                        }
                    },
                    "linux" : {
                        node('linux') {
                            stage('Clone sources') {
                                git url: 'git@github.com:rakudo/rakudo.git'
                                git url: 'git@github.com:perl6/nqp.git'
                                git url: 'git@github.com:MoarVM/MoarVM.git'
                            }

                            dir('MoarVM') {
                                sh 'perl Configure.pl'
                                sh 'make'
                                sh 'make install'
                            }
                            dir('nqp') {
                                sh 'perl Configure.pl --prefix="$WORKSPACE/MoarVM/install"'
                                sh 'make'
                                sh 'make install'
                            }
                            dir('rakudo') {
                                sh 'perl Configure.pl --prefix="$WORKSPACE/nqp/install"'
                                sh 'make'
                                sh 'make install'
                            }
                        }
                    }
                )                
            }
        }
    }    
}
