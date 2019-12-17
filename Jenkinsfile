import groovy.json.JsonOutput
def trigger_props
pipeline {
    agent { dockerfile true }
    environment {
	GIT_ACCESS_KEY = credentials('test-trigger-cred')
    }
    stages {
	stage('github pending status') {
	    steps {
		script {
		    data = JsonOutput.toJson([
			state: "pending",
			context: "continuous-integration/jenkins",
			description: "Jenkins",
			target_url: "${env.BUILD_URL}console"
		    ])
		}
		sh 'echo Event type: $x_github_event'
		sh 'echo Statuses URL: $GITHUB_STATUSES_URL'
		sh 'echo Build URL: $BUILD_URL'
		sh """
curl -X POST \
    "$GITHUB_STATUSES_URL?access_token=$GIT_ACCESS_KEY" \
    -H "Content-Type: application/json" \
    -d '${data}'
"""
	    }
	}
        stage('build') {
            steps {
		checkout([$class: 'GitSCM', branches: [[name: '*/master']],
			  userRemoteConfigs: [[url: 'https://github.com/cniles/test-trigger']]])
		script {
		    trigger_props = readYaml file: 'props.yml'
		    echo trigger_props.values.name
		}
            }
        }
	stage('changebranch') {
	    steps {
		script {
		    if (env.X_GitHub_Event == 'pull_request') {
			echo 'Is a change branch'
			echo env.pullRequest
		    } else {
			echo env.X_GitHub_Event
		    }
		}
	    }
	}
	stage('postbuild') {
	    steps {
		echo trigger_props.values.accountid.toString()
	    }
	}

	stage('github success status') {
	    steps {
		script {
		    data = JsonOutput.toJson([
			state: "success",
			context: "continuous-integration/jenkins",
			description: "Jenkins",
			target_url: "${env.BUILD_URL}console"
		    ])
		}
		sh 'echo Event type: $x_github_event'
		sh 'echo Statuses URL: $GITHUB_STATUSES_URL'
		sh 'echo Build URL: $BUILD_URL'
		sh """
curl -X POST \
    "$GITHUB_STATUSES_URL?access_token=$GIT_ACCESS_KEY" \
    -H "Content-Type: application/json" \
    -d '${data}'
"""
	    }
	}
    }
}

