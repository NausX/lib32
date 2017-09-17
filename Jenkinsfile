pipeline {
    agent any
    stages {
        stage('Prepare') {
            steps {
                sh '''
                    GIT_COMMIT=$(git rev-parse HEAD)
                    GIT_COMMIT_FILES=$(git show --pretty=format: --name-only ${GIT_COMMIT})
                    PACKAGE=none
                    for f in ${GIT_COMMIT_FILES[@]};do
                        if [[ "$f" == */PKGBUILD ]];then
                            PACKAGE=${f%/PKGBUILD}
                        fi
                    done
                    echo ${PACKAGE} > package.txt
                    case ${PACKAGE} in
                        'llvm') REPO='world' ;;
                        *) REPO='lib32' ;;
                    esac
                    case ${BRANCH_NAME} in
                        'testing'|'staging')
                            REPO=${REPO}-${BRANCH_NAME}
                            BUILDPKG="buildpkg-${BRANCH_NAME}"
                        ;;
                        'master')
                            BUILDPKG="buildpkg"
                        ;;
                        PR-*)
                            REPO=${REPO}-testing
                            BUILDPKG="buildpkg-testing"
                        ;;
                    esac
                    echo ${BUILDPKG} > cmd.txt
                    echo ${REPO} > repo.txt
                '''
            }
        }
        stage('Build') {
            environment {
                BUILDPKG = readFile('cmd.txt')
                REPO = readFile('repo.txt')
                PACKAGE = readFile('package.txt')
            }
            steps {
                sh '''
                    ${BUILDPKG} -p ${PACKAGE} -z ${REPO}
                '''
            }
            post {
                success {
                    sh '''
                        deploypkg -x -p ${PACKAGE} -r ${_REPO}
                    '''
                }
            }
        }
    }
}
