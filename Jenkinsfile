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
                    REPO='lib32'
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
                    ${BUILDPKG} -p ${PACKAGE} -z ${REPO} -a multilib
                '''
            }
            post {
                success {
                    sh '''
                        case ${PACKAGE} in
                            'llvm')
                                case ${BRANCH_NAME} in
                                    'testing'|'staging') _REPO=world-${BRANCH_NAME} ;;
                                    'master') _REPO='world' ;;
                                esac
                            ;;
                            *) _REPO=${REPO} ;;
                        esac
                        deploypkg -x -p ${PACKAGE} -r ${_REPO}
                    '''
                }
            }
        }
    }
}
