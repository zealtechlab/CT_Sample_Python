pipeline { 
    agent any
    options {
        ansiColor('xterm')
        skipStagesAfterUnstable()
        }

    stages {
        // CloneCode stage is commented as the repo is already cloned by the Jenkins pipe
        stage("CloneCode") {
            steps {
                script {
                    sh 'printenv'
                    // git %GIT_URL%;
                    stash(name: 'scm', includes: '*') 
                }
            }
        }
        stage("Selenium Testing") {
            steps {
                script {
                    sh "selenium-side-runner -s http://seleniumhub:4444/wd/hub \
                        SIDE/ZTL_Spring_Selemium_SIDE.side -c 'browserName=chrome' \
                        --output-directory=target/side --output-format=junit \
                        --base-url http://172.27.0.1:5000"
                }
            }
            post {
                always {
                    perfReport filterRegex: '', sourceDataFiles: '**target/side/*.xml'
                }
            }
        }
        stage("Jmeter Perf Testing") {
            steps {
                script {
                    sh "/opt/apache-jmeter-5.3/bin/jmeter -Jjmeter.save.saveservice.output_format=xml \
                        -Jjmeter.save.saveservice.response_data=true -Jjmeter.save.saveservice.samplerData=true \
                        -Jjmeter.save.saveservice.requestHeaders=true -Jjmeter.save.saveservice.url=true \
                        -Jjmeter.save.saveservice.responseHeaders=true \
                        -n -t 'perf/HTTP Request.jmx' \
                        -l target/perf/perfomancelog.jtl -j target/perf/jmeter.log"
                }
            }
            post {
                always {
                    perfReport filterRegex: '', sourceDataFiles: '**target/perf/*.jtl;**target/perf/*.xml'
                }
            }
        }
    }
    
    post {
        always {
            echo 'JENKINS PIPELINE - $env.BUILD_URL, ${env.BUILD_NUMBER}'
        }
        notBuilt {
            echo '${env.BUILD_NUMBER} ${env.BUILD_URL} JENKINS PIPELINE NOT BUILT, STOPPED at STAGE - ${env.STAGE_NAME}'
        }
        success {
            echo '${env.BUILD_NUMBER} ${env.BUILD_URL} JENKINS PIPELINE SUCCESSFUL'
        }
        failure {
            echo '${env.BUILD_NUMBER} ${env.BUILD_URL} JENKINS PIPELINE FAILED at Stage - ${env.STAGE_NAME}'
        }
        unstable {
            echo '${env.BUILD_NUMBER} ${env.BUILD_URL} JENKINS PIPELINE WAS MARKED AS UNSTABLE'
        }
        changed {
            echo '${env.BUILD_NUMBER} ${env.BUILD_URL} JENKINS PIPELINE STATUS HAS CHANGED SINCE LAST EXECUTION'
        }
        aborted {
            echo '${env.BUILD_NUMBER} ${env.BUILD_URL} JENKINS PIPELINE ABORTED'
        }
    }
}