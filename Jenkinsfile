// pipeline {
//     agent any

//     parameters {
//         string(name: 'IMAGE_TAG', defaultValue: '30', description: 'Which version to deploy?')
//         string(name: 'LAST_GOOD_BUILD', defaultValue: '47', description: 'Last known good build number')
//     }

//     stages {
//         // stage('dummy'){
//         //     steps{
//         //         sh 'echo '
//         //     }
//         // }
//         stage('Load Library') {
//             steps {
//                 script {
//                     def libBranch = env.BRANCH_NAME 
//                       echo "Loading library branch: ${libBranch}"

//                      library "my-jenkins-lib@${libBranch}"
//                 }
            
//             }
//         }
//         stage('Checkout') {
//             steps {
//                 git url: 'https://github.com/gvyshnaviyadav-ops/full_sd.git', branch: "main"
//             }
//         }

//         stage('Build') {
//             steps {
//                 script {
//                     withCredentials([usernamePassword(
//                         credentialsId: 'docker-registry-creds',
//                         usernameVariable: 'DOCKER_USER',
//                         passwordVariable: 'DOCKER_PASS'
//                     )]) {
//                         sh 'echo $DOCKER_PASS | docker login -u $DOCKER_USER --password-stdin'
//                         buildApp("${IMAGE_TAG}", "$DOCKER_USER")
//                     }
//                 }
//             }
//         }
//           stage('security scan'){
//             steps{
//                 script{
//                      withCredentials([usernamePassword(
//                 credentialsId: 'docker-registry-creds',
//                 usernameVariable: 'DOCKER_USER',
//                 passwordVariable: 'DOCKER_PASS'
//             )]) {
//                 sh "echo $DOCKER_PASS | docker login -u $DOCKER_USER --password-stdin"
//               sh '''
//               docker run  -v /var/run/docker.sock:/var/run/docker.sock -v \${WORKSPACE}:/workspace --rm aquasec/trivy image --severity CRITICAL --exit-code 0 --format json  --output /workspace/trivy-frontend.json $DOCKER_USER/todo-frontend:${IMAGE_TAG}
//             docker run  -v /var/run/docker.sock:/var/run/docker.sock -v \${WORKSPACE}:/workspace --rm aquasec/trivy image --severity CRITICAL --exit-code 0 --format json --output /workspace/trivy-backend.json  $DOCKER_USER/todo-backend:${IMAGE_TAG}
//             '''
//               //sh 'ls -la trivy-*.json'
//                 }
//                 }
//             }
//              }
      
        

//         stage('Deploy') {
//             steps {
//                 script {
                    
//                         withCredentials([usernamePassword(
//                             credentialsId: 'docker-registry-creds',
//                             usernameVariable: 'DOCKER_USER',
//                             passwordVariable: 'DOCKER_PASS'
//                         )]) {
//                             deployApp("todoapp", "${params.IMAGE_TAG}", "${DOCKER_USER}")
                        
//                     } 
//                 }
//             }
//         }
        
//         stage('Health Checks') {
           
//             steps {
//                 script {
//                     sh "sleep 45"
//                     def backendStatus = sh(
//                 script: "docker ps --filter 'name=todoapp_backend' --format '{{.Status}}' | head -n 1",
//                 returnStdout: true
//             ).trim()

//             echo "Backend Status: ${backendStatus}"

//             if (!backendStatus.contains("healthy")) {
//                 error("Backend container is not healthy")
//             }

        
//             def dbStatus = sh(
//                 script: "docker ps --filter 'name=todoapp_db' --format '{{.Status}}' | head -n 1",
//                 returnStdout: true
//             ).trim()


//             if (!dbStatus.contains("healthy")) {
//                 error("Database container is not healthy")
//             }

//             def nginxStatus = sh(
//                 script: "docker ps --filter 'name=todoapp_nginx' --format '{{.Status}}' | head -n 1",
//                 returnStdout: true
//             ).trim()

          
//             if (!nginxStatus.contains("healthy")) {
//                 error("Nginx container is not healthy")
//             }

//                 }
//             }
            
//         }
             
//             stage('tag and push'){
//                 steps{
//                   script {
//                     withCredentials([usernamePassword(
//                         credentialsId: 'docker-registry-creds',
//                         usernameVariable: 'DOCKER_USER',
//                         passwordVariable: 'DOCKER_PASS'
//                     )]) {
//                         sh 'echo $DOCKER_PASS | docker login -u $DOCKER_USER --password-stdin'
//                         tagPush("${IMAGE_TAG}", "$DOCKER_USER")
//                     }
//                 }  
//                 }
//             }
//         }
        
    

//     post {
//         failure {
//             script {
//                 echo "Pipeline failed, rolling back..."
//                 withCredentials([usernamePassword(
//                     credentialsId: 'docker-registry-creds',
//                     usernameVariable: 'DOCKER_USER',
//                     passwordVariable: 'DOCKER_PASS'
//                 )]) {
//                     echo "Rolling back "
//                          sh "docker pull ${DOCKER_USER}/todo-frontend:${params.LAST_GOOD_BUILD}"
//                         sh "docker pull ${DOCKER_USER}/todo-backend:${params.LAST_GOOD_BUILD}"
//                         fdeployApp("todoapp", "${params.LAST_GOOD_BUILD}", "${DOCKER_USER}")
//                 }
//             }
//         }

//         always {
//             script{
//              notifyC('webhook',env.JOB_NAME,env.BUILD_NUMBER,env.BUILD_URL,  currentBuild.currentResult)
//              sh 'ls -lah'
//              archiveArtifacts artifacts: 'trivy-*.json', allowEmptyArchive: true
//         }
//         }
//     }
// }

@Library('my-jenkins-lib@main') _

pipeline {
    agent any

    environment {
        CURRENT_STAGE = "NONE"
    }

    parameters {
        string(name: 'IMAGE_TAG', defaultValue: '30', description: 'Which version to build?')
        string(name: 'LAST_GOOD_BUILD', defaultValue: '47', description: 'Last known good build number')
        string(name: 'BRANCH_NAME', defaultValue: 'main', description: 'which branch of git')
    }

    stages {

        stage('Checkout') {
            steps {
                script {
                    env.CURRENT_STAGE = "checkout"

                    try {
                        git url: 'https://github.com/gvyshnaviyadav-ops/full_sd.git',
                            branch: "${params.BRANCH_NAME}"
                    }
                    catch (Exception e) {
                        error("gitlogin failed: ${e.message}")
                    }
                }
            }
        }

        stage('Build') {
            steps {
                script {
                    env.CURRENT_STAGE = "build"

                    withCredentials([usernamePassword(
                        credentialsId: 'docker-registry-creds',
                        usernameVariable: 'DOCKER_USER',
                        passwordVariable: 'DOCKER_PASS'
                    )]) {

                        try {
                            sh 'echo $DOCKER_PASS | docker login -u $DOCKER_USER --password-stdin'
                         
                        }
                        catch (Exception e) {
                            error("Docker login failed: ${e.message}")
                        }
                            try {
                            buildApp("${params.IMAGE_TAG}", "${DOCKER_USER}")
                        }
                        catch (Exception e) {
                            error("Build failed: ${e.message}")
                        }

                       
                    }
                }
            }
        }

        stage('Security Scan') {
            steps {
                script {
                    env.CURRENT_STAGE = "scan"

                    withCredentials([usernamePassword(
                        credentialsId: 'docker-registry-creds',
                        usernameVariable: 'DOCKER_USER',
                        passwordVariable: 'DOCKER_PASS'
                    )]) {

                        try {
                            sh 'echo $DOCKER_PASS | docker login -u $DOCKER_USER --password-stdin'
                        }
                        catch (Exception e) {
                            error("Docker login failed: ${e.message}")
                        }

                        try {

                            sh """
                            trivy image \
                            --severity HIGH,CRITICAL \
                            --format table \
                            -o trivy-frontend.json \
                            --exit-code 0 \
                            ${DOCKER_USER}/todo-frontend:${params.IMAGE_TAG}
                            """

                            sh """
                            trivy image \
                            --severity HIGH,CRITICAL \
                            --format table \
                            -o trivy-backend.json \
                             --exit-code 0\
                            ${DOCKER_USER}/todo-backend:${params.IMAGE_TAG}
                            """

                         


                      

                        }
                        catch (Exception e) {

                            error("""
Image is vulnerable.

High/Critical vulnerabilities found.

Check:
- trivy-frontend.json
- trivy-backend.json

${e.message}
""")

                        }
                    }
                }
            }
        }

        stage('Push Images') {
            steps {
                script {

                    env.CURRENT_STAGE = "push"

                    withCredentials([usernamePassword(
                        credentialsId: 'docker-registry-creds',
                        usernameVariable: 'DOCKER_USER',
                        passwordVariable: 'DOCKER_PASS'
                    )]) {

                        try {

                            sh """
                            docker push ${DOCKER_USER}/todo-frontend:${params.IMAGE_TAG}
                            docker push ${DOCKER_USER}/todo-backend:${params.IMAGE_TAG}
                            """

                        }
                        catch (Exception e) {

                            error("Push failed: ${e.message}")

                        }
                    }
                }
            }
        }

        stage('Trigger CD') {
            steps {
                script {

                    build(
                        job: 'todo-cd',
                        wait: false,
                        parameters: [
                            string(
                                name: 'IMAGE_TAG',
                                value: params.IMAGE_TAG
                            ),
                            string(
                                name: 'LAST_GOOD_BUILD',
                                value: params.LAST_GOOD_BUILD
                            ),
                            string(
                                name: 'BRANCH_NAME',
                                value: params.BRANCH_NAME
                            )
                        ]
                    )

                }
            }
        }
    }

    post {

        always {

            script {

                notifyC(
                    'webhook',
                    env.JOB_NAME,
                    env.BUILD_NUMBER,
                    env.BUILD_URL,
                    currentBuild.currentResult,
                    env.CURRENT_STAGE
                )

                archiveArtifacts artifacts: 'trivy-*.json',
                                 allowEmptyArchive: true
            }
        }
    }
}