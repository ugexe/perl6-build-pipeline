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
                    "windows" : {
                        node('windows') {
                            bat 'mkdir %WORKSPACE%/report/nqp'
                            bat 'mkdir %WORKSPACE%/report/rakudo'

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
                                    prove --archive "%WORKSPACE%/report/nqp" --timer -v -r t/
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
                                    prove --archive "%WORKSPACE%/report/rakudo" --timer -v -r t/
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
                            sh 'mkdir -p $WORKSPACE/report/nqp'
                            sh 'mkdir -p $WORKSPACE/report/rakudo'

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
                                sh 'prove --archive "$WORKSPACE/report/nqp" --timer -v -r t/'
                                sh 'make install'
                            }
                            dir('rakudo') {
                                git url: 'https://github.com/rakudo/rakudo.git'
                                sh 'perl Configure.pl --prefix="$WORKSPACE/install"'
                                sh 'make'
                                sh 'prove --archive "$WORKSPACE/report" --timer -v -r t/''
                                sh 'make install'
                            }
                        }
                    }
                )
            }
        }
    }    
}
