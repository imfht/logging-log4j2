#!groovy
/*
 * Licensed to the Apache Software Foundation (ASF) under one or more
 * contributor license agreements.  See the NOTICE file distributed with
 * this work for additional information regarding copyright ownership.
 * The ASF licenses this file to You under the Apache License, Version 2.0
 * (the "License"); you may not use this file except in compliance with
 * the License.  You may obtain a copy of the License at
 *
 *     https://www.apache.org/licenses/LICENSE-2.0
 *
 * Unless required by applicable law or agreed to in writing, software
 * distributed under the License is distributed on an "AS IS" BASIS,
 * WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
 * See the License for the specific language governing permissions and
 * limitations under the License.
 */

pipeline {
    options {
        ansiColor('xterm')
        buildDiscarder logRotator(numToKeepStr: '10')
        timeout(90)
    }
    agent none
    stages {
        stage('Build') {
            failFast true
            parallel {
                stage('Ubuntu') {
                    agent { label 'ubuntu' }
                    steps {
                        withMaven(jdk: 'JDK 1.8 (latest)', maven: 'Maven 3 (latest)') {
                            sh 'mvn -B -fae -t toolchains-jenkins-ubuntu.xml -Djenkins -V clean install site'
                        }
                    }
                    post {
                        always {
                            recordIssues enabledForFailure: true, tool: mavenConsole(), referenceJobName: 'log4j/master'
                            recordIssues enabledForFailure: true, tool: errorProne(), referenceJobName: 'log4j/master'
                            recordIssues enabledForFailure: true, tools: [java(), javaDoc()], sourceCodeEncoding: 'UTF-8', referenceJobName: 'log4j/master'
                            recordIssues tools: [cpd(), checkStyle(), pmdParser(), spotBugs()], sourceCodeEncoding: 'UTF-8', referenceJobName: 'log4j/master'
                        }
                    }
                }
                stage('Windows') {
                    agent { label 'Windows' }
                    steps {
                        bat 'if exist %userprofile%\\.embedmongo\\ rd /s /q %userprofile%\\.embedmongo'
                        withMaven(jdk: 'JDK 1.8 (latest)', maven: 'Maven 3 (latest)') {
                            bat 'mvn -B -fae -t toolchains-jenkins-win.xml -V -Dfile.encoding=UTF-8 clean install'
                        }
                    }
                }
            }
        }
    }
    post {
        failure {
            slackSend channel: 'logging', message: "Jenkins build failure: ${env.BUILD_URL}"
        }
    }
}
