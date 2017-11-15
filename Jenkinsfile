pipeline {
    agent none
    options {
        timestamps()
        timeout(time: 60, unit: 'MINUTES')
    }
    stages {
        stage("distribute") {
            steps {
                parallel (
                    "windows" : {
                        node('windows') {
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
                                bat '''
                                    call "C:\\Program Files (x86)\\Microsoft Visual Studio\\2017\\Community\\VC\\Auxiliary\\Build\\vcvars64.bat"
                                    nmake test
                                '''
                                bat '''
                                    call "C:\\Program Files (x86)\\Microsoft Visual Studio\\2017\\Community\\VC\\Auxiliary\\Build\\vcvars64.bat"
                                    nmake install
                                '''
                            }
                        }
                    },
                    "linux" : {
                        node('linux') {
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
