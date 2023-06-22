def Deploy(DeployEnv) {
    sh """
    source ../../assume_role.sh
    helm upgrade flask helm/ --atomic --wait --install --namespace "$DeployEnv" --create-namespace --set deployment.tag="$GIT_COMMIT" --set deployment.env="$DeployEnv"
    """
}

pipeline {
    agent any
    environment {
        image_name="699509601278.dkr.ecr.eu-central-1.amazonaws.com/flask"
        region="eu-central-1"
        account="699509601278"
    }
    stages {
        stage("Build") {
            steps {
                sh '''
                docker build -t "${image_name}:$GIT_COMMIT" .
                '''
            }
        }
        stage("test") {
            steps {
                sh '''
                docker run -dit -p 5000:5000 "${image_name}:$GIT_COMMIT" || docker stop $(docker ps -a -q)
                sleep 10
                curl http://localhost:5000
                exit_status=$?
                if [[ $exit_status == 0 ]]
                then echo "SUCCESFULL TESTS" && docker stop $(docker ps -a -q)
                else echo "TESTS FAILED" && docker stop $(docker ps -a -q) && exit 1
                fi
                '''
            }
        }
        stage("Push") {
            steps {
                sh '''
                aws ecr get-login-password --region eu-central-1 | docker login --username AWS --password-stdin 699509601278.dkr.ecr.eu-central-1.amazonaws.com
                docker push ${image_name}:$GIT_COMMIT
                '''
            }
        }
        stage("Deploy_Dev") {
            when {
                expression {
                    env.BRANCH_NAME == "development"
                }
            }
            steps {
                Deploy("dev")
            }
        }
        stage("Deploy_Prod"){
            when {
                expression {
                    env.BRANCH_NAME == "master"
                }
            }
            steps {
                Deploy("prod")
            }
        }
        stage("Deploy_Stage"){
            when {
                expression {
                    env.BRANCH_NAME == "stage"
                }
            }
            steps {
                Deploy("stage")
            }
        }
    }
}