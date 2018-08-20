import groovy.json.JsonOutput
def TAG_ENV

node {




    parameters {

        string(name: 'APP_NAME', description: 'Short app name to be deployed')

        string(name: 'BUILD_TAG', description: 'Full docker image path')

        string(name: 'SLACK_URL', description: 'Full docker image path')

        string(name: 'CHANNEL', description: 'Full docker image path', 'default': '#notify-dev')

        string(name: 'QENV', description: 'Deploy Environment name ')

        string(name: 'REMOTE_DEV_HOST', description: 'Remote dev servers (ssh url format), now supports single remote server.')

    }




    try {

        stage("Get docker images") {

            checkout scm

            env.PATH = "$env.PATH:/usr/local/bin"

            sh(script: 'git show --name-status', returnStdout: true).trim()




            echo ">> APP_NAME: $params.APP_NAME"

            echo ">> BUILD_TAG: $params.BUILD_TAG"
}
}
}

