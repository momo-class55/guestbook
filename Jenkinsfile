import java.text.SimpleDateFormat
def TODAY = (new SimpleDateFormat("yyyyMMddHHmmss")).format(new Date())
pipeline {
    agent any
    environment {
     strDockerTag = "${TODAY}_${BUILD_ID}"
     strDockerImage ="misun1234/cicd_guestbook:${strDockerTag}"
    }
    stages {
        stage('Checkout') {
            steps {
                echo 'Checkout'
                git(branch:'master'
                    , url:'https://github.com/momo-class55/guestbook.git')
            }
        }
        
        stage('Build'){
            steps {
                sh './mvnw clean package'
            }
            
        }
        
        stage('Unit Test'){
            steps {
                sh './mvnw test'
            }
            
            post {
                always {
                    junit '**/target/surefire-reports/TEST-*.xml'
                }
            }
        }
        
        stage('SonarQube Analysis') {
            steps{
                withSonarQubeEnv('SonarQube-Server'){
                    sh '''
                        ./mvnw sonar:sonar \
                        -Dsonar.projectKey=guestbook
                    '''
                }
            }
        }
        stage('SonarQube Quality Gate'){
            steps{
                timeout(time: 2, unit: 'MINUTES') {
                    script{
                        def qg = waitForQualityGate()
                        if(qg.status != 'OK') {
                            echo "NOT OK Status: ${qg.status}"
                            error "Pipeline aborted due to quality gate failure: ${qg.status}"
                        } else{
                            echo "OK Status: ${qg.status}"
                        }
                    }
                }
            }
        }
        
        stage('Docker Image Build') {
             steps {
                     script {
                     
                     oDockImage = docker.build(strDockerImage, "--no-cache --build-arg VERSION=${strDockerTag} -f Dockerfile .")
                     }
             }
        }
        
          stage('Docker Image Push') {
             steps {
                 script {
                     docker.withRegistry('', 'DockerHub_Credential') {
                        oDockImage.push()
                     }
                }
            }
        }
          
          stage('Staging Deploy') {
             steps {
                 sshagent(credentials: ['Staging-PrivateKey']) {
                     sh "ssh -o StrictHostKeyChecking=no root@43.203.218.87 docker container rm -f guestbookapp"
                     sh "ssh -o StrictHostKeyChecking=no root@43.203.218.87 docker container run \
                     -d \
                     -p 38080:80 \
                     --name=guestbookapp \
                     -e MYSQL_IP=172.31.0.100 \
                     -e MYSQL_PORT=3306 \
                     -e MYSQL_DATABASE=guestbook \
                     -e MYSQL_USER=root \
                     -e MYSQL_PASSWORD=education \
                     ${strDockerImage} "
                 }
             }
         }
        
    }
    
    
    
    post { 
    
     always { 
    slackSend(tokenCredentialId: 'slack-token'
     , channel: '#교육'
     , color: 'good'
     , message: "${JOB_NAME} (${BUILD_NUMBER}) 끝 Details: (<${BUILD_URL} | here >)")
     }
     success { 
    slackSend(tokenCredentialId: 'slack-token'
     , channel: '#교육'
     , color: 'good'
     , message: "${JOB_NAME} (${BUILD_NUMBER}) 성공 Details: (<${BUILD_URL} | here >)")
     }
     failure { 
    slackSend(tokenCredentialId: 'slack-token'
     , channel: '#교육'
     , color: 'danger'
     , message: "${JOB_NAME} (${BUILD_NUMBER}) 실패 Details: (<${BUILD_URL} | here >)")
     }
     }
    
    
    
    
}

