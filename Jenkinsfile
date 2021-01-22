#!groovy
@Library('huddlygo_shared_libraries@master') _

Map stage_node_info = [:]
String slack_channel = BRANCH_NAME == "master" ? "#builds-falcon" : ""

pipeline {
    agent none

    environment {
        REMOTE="falcon"
        ARTIFACTORY_ACCESS_TOKEN=credentials('artifactory-access-token')
        ARTIFACTORY_USER="jenkins"
    }

    stages {
        stage ('package modules') {
            agent { docker huddlydocker([
                configKey: "falcon_pytooling_build",
                label: "docker"
            ]) }
            stages {
                stage ('build & upload') {
                    when {
                        branch 'develop'
                    }
                    steps {
                        sh script: "python3 setup.py sdist upload -r $REMOTE", label: "Build and upload conan pip package to artifactory"
                    }
                    post {
                        always {
                            script { stage_node_info["$STAGE_NAME"] = "$NODE_NAME"}
                        }
                    }
                }
            }
        }
    }
    post {
        failure {
            send_slack_Failure(BRANCH_NAME, slack_channel, stage_node_info)
            setBuildDescription(stage_node_info)
        }
        aborted {
            setBuildDescription(stage_node_info)
        }
    }
}
