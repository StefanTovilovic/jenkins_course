def sendEmailFuncion(name, id, test_results) {
    echo (message: "Sending email")
    echo (message: "Name: $name, Id: $id, Results: $test_results")
}
pipeline {
    agent any
    
    parameters {
        string defaultValue: 'pipeline', name: 'GIT_BRANCH'
        booleanParam defaultValue: true, name: 'RUN_TEST'
        booleanParam defaultValue: true, name: 'SEND_EMAIL'
    }
    
    stages {
        stage('Download') {
            steps {
                cleanWs()
                echo (message: "Download")
                dir('pipeline') {
                    git (
                        branch: "${params.GIT_BRANCH}",
                        url: 'https://github.com/KLevon/jenkins-course'
                    )
                }
                rtDownload (
                    serverId: 'artifactory',
                    spec: '''{
                        "files": [
                            {
                                "pattern": "generic-local/printer.zip",
                                "target": "example-repo-local/printer.zip"
                            }
                        ]
                    }'''
                    
                )
                unzip (
                    zipFile: "example-repo-local/printer.zip",
                    dir: "pipeline/"
                )
            }
        }
        stage('Build') {
            steps {
                echo (message: "Build")
                withCredentials (
                    [usernamePassword(credentialsId: 'CRED_NAME', passwordVariable: 'psw', usernameVariable: 'usr')]
                ) {
                    echo (message: "username: $usr, password: $psw")
                }
                bat (
                    script: """
                        cd pipeline
                        Makefile.bat
                    """
                )
            }
        }
        stage('Publish') {
            steps {
                echo (message: "Publish")
                script {
                    zip (
                        zipFile: "pipeline.zip",
                        archive: true,
                        dir: "pipeline"
                    )
                }
                rtUpload (
                    serverId: 'artifactory',
                    spec: """{
                        "files": [
                            {
                                "pattern": "pipeline.zip",
                                "target": "generic-local/release/${env.BUILD_ID}/"
                            }
                        ]
                    }"""
                    
                )
            }
        }
        stage('Tests') {
            when {
                equals expected: true,
                actual: params.RUN_TEST
            }
            steps {
                echo (message: "Tests")
                script {
                    env.output = ''
                    def modules = ['printer', 'scanner', 'main']
                    for (module in modules) {
                        env.output += bat (
                            script: """
                                cd pipeline
                                Tests.bat $module
                            """, returnStdout: true
                        ).trim()
                    }
                }
            }
        }
        stage('Dynamic') {
            when {
                branch 'feature/multi/*'
            }
            steps {
                echo (message: "New Stage")
            }
        }
    }
    post {
        always {
            script {
                if (params.SEND_EMAIL == true)
                    sendEmailFuncion(env.JOB_NAME, env.BUILD_ID, env.output)
            }
        }
    }
}
