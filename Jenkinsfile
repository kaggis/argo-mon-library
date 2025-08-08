pipeline {
    agent any
    options {
        checkoutToSubdirectory('argo-mon-library')
    }
    environment {
        PROJECT_DIR="argo-mon-library"
        GIT_COMMIT=sh(script: "cd ${WORKSPACE}/$PROJECT_DIR && git log -1 --format=\"%H\"",returnStdout: true).trim()
        GIT_COMMIT_HASH=sh(script: "cd ${WORKSPACE}/$PROJECT_DIR && git log -1 --format=\"%H\" | cut -c1-7",returnStdout: true).trim()
        GIT_COMMIT_DATE=sh(script: "date -d \"\$(cd ${WORKSPACE}/$PROJECT_DIR && git show -s --format=%ci ${GIT_COMMIT_HASH})\" \"+%Y%m%d%H%M%S\"",returnStdout: true).trim()

    }
    stages {
        stage ('Check and Test...') {
            parallel {              
                stage('Docker env with rocky9 image') {
                    agent {
                        docker {
                            image 'argo.registry:5000/epel-9-acc'
                            args '-u jenkins:jenkins'
                        }
                    }
                    stages {
                        stage ('Run Linting') {
                            steps {
                                echo 'Executing lint checks'
                                sh '''
                                    cd ${WORKSPACE}/$PROJECT_DIR
                                    pip install flake8 isort bandit mypy

                                    export PATH=$HOME/.local/bin:$PATH

                                    echo "Running flake8..."
                                    flake8 .

                                    echo "Running isort..."
                                    isort --check-only .

                                    echo "Running bandit..."
                                    bandit -r .

                                    echo "Running mypy..."
                                    mypy .
                                '''
                            }
                        }
                        stage ('Run tests') {
                            steps {
                                echo 'Executing tox tests'
                                sh '''
                                    cd ${WORKSPACE}/$PROJECT_DIR
                                    rm -f .python-version &>/dev/null
                                    rm -rf .coverage* .tox/ coverage.xml &> /dev/null
                                    source $HOME/pyenv.sh
                                    ALLPYVERS=$(pyenv versions | grep '^[ ]*[0-9]' | tr '\n' ' ')
                                    echo Found Python versions $ALLPYVERS
                                    pyenv local $ALLPYVERS
                                    export TOX_SKIP_ENV="py27.*|py36.*"
                                    tox -p all
                                    coverage xml --omit=*usr* --omit=*.tox*
                                '''
                                cobertura coberturaReportFile: '**/coverage.xml'
                            }
                        }
                    }
                }
            }
        }
    }
    post {
        always {
            echo 'Cleaning workspace and exiting'
            cleanWs()
        }
    }
}