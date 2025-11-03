pipeline {
    agent any

    environment {
        // 공통 PYTHONPATH (한 경로만 지정하므로 OS별 path separator 차이 영향 없음)
        PYTHONPATH = "${WORKSPACE}"
        // 로그 버퍼링 최소화(선택)
        PYTHONUNBUFFERED = "1"
    }

    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Set Up Python Environment') {
            steps {
                script {
                    if (isUnix()) {
                        sh '''
                            set -e

                            # python3 우선, 없으면 python
                            (command -v python3 >/dev/null 2>&1 && python3 -m venv venv) || python -m venv venv

                            . venv/bin/activate
                            pip install --upgrade pip

                            # requirements.txt가 있으면 설치, 없으면 기본 툴(pytest) 확보
                            if [ -f requirements.txt ]; then
                                pip install -r requirements.txt
                            else
                                pip install pytest
                            fi
                        '''
                    } else {
                        bat '''
                            @echo off
                            rem py(Windows Python Launcher) 우선, 없으면 python 사용
                            py -3 -m venv venv || python -m venv venv

                            call venv\\Scripts\\activate.bat
                            python -m pip install --upgrade pip

                            if exist requirements.txt (
                                pip install -r requirements.txt
                            ) else (
                                pip install pytest
                            )
                        '''
                    }
                }
            }
        }

        stage('Run pytest') {
            steps {
                script {
                    if (isUnix()) {
                        sh '''
                            set -e
                            . venv/bin/activate
                            # JUnit XML 리포트 생성 (테스트 폴더명은 필요에 맞게 조정)
                            pytest tests/ --junitxml=pytest-report.xml
                        '''
                    } else {
                        bat '''
                            @echo off
                            call venv\\Scripts\\activate.bat
                            pytest tests/ --junitxml=pytest-report.xml
                        '''
                    }
                }
            }
        }
    }

    post {
        always {
            // 리포트 게시 (파일이 없더라도 스테이지는 실행됨)
            junit allowEmptyResults: true, testResults: 'pytest-report.xml'
        }
        success {
            echo '테스트 자동화가 성공적으로 완료되었습니다!'
        }
        failure {
            echo '테스트 자동화 중 일부 테스트가 실패했습니다. 리포트를 확인해주세요.'
        }
    }
}
