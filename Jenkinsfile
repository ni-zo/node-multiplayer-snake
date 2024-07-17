pipeline {
    agent any
    environment {
        SNYK_HOME = tool name:'security-analysis'
    }
    
    parameters{
    choice choices: ["Baseline","APIS","FULL"],
    description: 'Type of scan that is going to perform inside the container',
    name: 'SCAN_TYPE'
    
    string defaultValue:"http://172.19.0.2:3000",
    description: 'Target URL to scan',
    name:'TARGET'
    }
    
    stages {
        stage('Build'){
            steps {
                echo 'Building...'
                git 'https://github.com/ni-zo/node-multiplayer-snake.git'
                sh 'npm i'
                sh 'docker build -t app-test .'
            }
        }
    
    /*stage('SAST'){
            steps {
                withCredentials([string(credentialsId: 'SNYK_TOKEN_TEXT', variable:'SNYK_TOKEN_TEXT')]){
                    echo 'Analysing source code...'
                    sh "${SNYK_HOME}/snyk-linux auth ${SNYK_TOKEN_TEXT}"
                    sh "${SNYK_HOME}/snyk-linux code test --debug"
                }
            }*/
            
            stage('SCA'){
                steps{
                    echo 'Analysing software components...'
                    snykSecurity(
                        snykInstallation: 'security-analysis',
                        snykTokenId: 'SNYK_TOKEN',
                        failOnIssues: 'false'
                    )
                }
            }
    
    
    stage('Tests'){
        steps {
            echo 'Testing...'
        }
    }
    
    /*stage('Deploy') {
        steps {
            echo 'Deploying...'
            //sh 'docker container stop app-test'
            sh '''
            docker run --rm --net zapnet --name app-test -dp 127.0.0.1:3000:3000 app-test 
            docker exec app-test ip a
            '''
        }
    }*/

    
   /* stage('DAST') {
  steps {
  echo 'Analyzing...'
  sh '''
  docker pull owasp/zap2docker-stable
  docker run -dt -p 8090:8090 --net zapnet --name owasp owasp/zap2docker-stable /bin/bash
  docker exec owasp mkdir /zap/wrk
  '''
  script {
  if(SCAN_TYPE.equals("Baseline")) {
  sh """
  docker exec owasp zap-baseline.py \
  -t ${params.TARGET} -x report.xml -I
  """
  } else if (SCAN_TYPE.equals("APIS")) {
    sh """
    docker exec owasp zap-api-scan.py \
    -t ${params.TARGET} -x report.xml -I
    """
  } else if (SCAN_TYPE.equals("Full")){
    sh """
    docker exec owasp zap-full-scan.py \
    -t ${params.TARGET} -x report.xml -I
    """
  }else{
    echo "something went wrong..."
  }
  }
  sh 'docker cp owasp:/zap/wrk/report.xml ${WORKSPACE}/report.xml'
  }
  
}*/
    
 /*   stage('DAST') {
  steps {
  echo 'Analyzing...'
  sh '''
  docker pull zaproxy/zap-stable
  docker run -dt -p 8090:8090 --net zapnet --name owasp zaproxy/zap-stable /bin/bash
  docker exec owasp mkdir /zap/wrk
  '''
  script {
  if(SCAN_TYPE.equals("Baseline")) {
  sh """
  docker exec owasp zap-baseline.py \
  -t ${params.TARGET} -x report.xml -I
  """
  } else if (SCAN_TYPE.equals("APIS")) {
    sh """
    docker exec owasp zap-api-scan.py \
    -t ${params.TARGET} -x report.xml -I
    """
  } else if (SCAN_TYPE.equals("Full")){
    sh """
    docker exec owasp zap-full-scan.py \
    -t ${params.TARGET} -x report.xml -I
    """
  }else{
    echo "something went wrong..."
  }
    sh 'docker cp owasp:/zap/wrk/report.xml ${WORKSPACE}/report.xml'
    //sh 'docker cp owasp:/zap/wrk/report.xml /var/lib/jenkins/workspace/Project\ SecDevOps/report.xml'
  }
  }
  
}*/
    stage('DAST') {
  steps {
  echo 'Analyzing...'
  sh '''
  docker pull zaproxy/zap-stable
  docker run -dt -p 8090:8090 --net zapnet --name owasp zaproxy/zap-stable /bin/bash
  docker exec owasp mkdir /zap/wrk
  '''
  script {
  if(SCAN_TYPE.equals("Baseline")) {
  sh """
  docker exec owasp zap-baseline.py \
  -t ${params.TARGET} -r report.html -I
  """
  } else if (SCAN_TYPE.equals("APIS")) {
    sh """
    docker exec owasp zap-api-scan.py \
    -t ${params.TARGET} -r report.html -I
    """
  } else if (SCAN_TYPE.equals("Full")){
    sh """
    docker exec owasp zap-full-scan.py \
    -t ${params.TARGET} -r report.html -I
    """
  }else{
    echo "something went wrong..."
  }
    sh 'docker cp owasp:/zap/wrk/report.html ${WORKSPACE}/report.html'
    //sh 'docker cp owasp:/zap/wrk/report.xml /var/lib/jenkins/workspace/Project\ SecDevOps/report.xml'
  }
  }
  
}

stage('Container Security') {
  steps {
    script {
      sh "trivy --format template --template \"@/opt/trivy/template/html.tpl\" -o report.html image app-test"
    }
  }
}
    
    }
    
    
    
    post {
  always {
    echo "Removing container"
    sh '''
    docker stop owasp
    docker rm owasp
    '''
      archiveArtifacts artifacts: "report.html", fingerprint: true
    
    publishHTML (target: [
      allowMissing: false,
      alwaysLinkToLastBuild: false,
      keepAll: true,
      reportDir: '.',
      reportFiles: 'report.html',
      reportName: 'Trivy Scan',
    ])
  }
}
}
