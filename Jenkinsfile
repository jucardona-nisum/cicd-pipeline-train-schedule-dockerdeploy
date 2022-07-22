pipeline {
    agent any
    stages {
        stage('Build') {
            steps {
                echo 'Running build automation'
                sh './gradlew build --no-daemon'
                archiveArtifacts artifacts: 'dist/trainSchedule.zip'
            }
        }
        stage('Build Docker Image'){
            when{
                branch 'master'
            }
            steps{
                script{
                    app = docker.build("nachocardona/train-schedule")
                    app.inside{
                        sh 'echo $(curl localhost:8080)'
                    }
                }
            }
        }
        stage("Push Docker Images"){
            when{
                branch 'master'
            }
            steps{
                script{
                    docker.withRegistry('https://registry.hub.docker.com', 'docker_hub'){
                        app.push("${env.BUILD_NUMBER}")
                        app.push("latest")
                    }
                }
            }
        }
        stage("Deploy to Production"){
            when{
                branch 'master'
            }
            steps{
                input 'Deploy to Production?'
                milestone(1)
                withCredentials([usernamePassword(credentialsId: 'webserver_login', usernameVariable: 'USERNAME', passwordVariable: 'PASSWORD')]) {
                  // available as an env variable, but will be masked if you try to print it out any which way
                  // note: single quotes prevent Groovy interpolation; expansion is by Bourne Shell, which is what you want
                  sh '1. echo $PASSWORD'
                  // also available as a Groovy variable
                  echo USERNAME
                  // or inside double quotes for string interpolation
                  echo "2. username is $USERNAME"
                }
                withCredentials ([usernamePassword(credentiaslId: 'webserver_login', usernameVariable: 'USERNAME', passwordVariable: 'PASSWORD' )]){
                    script{
                        echo "sshpass -p '$PASSWORD' -v ssh -o StrictHostKeyChecking=no $USERNAME@${env.prod_id} \"docker pull nachocardona/train-schedule:${env.BUILD_NUMBER}\" "
                        sh "sshpass -p '$PASSWORD' -v ssh -o StrictHostKeyChecking=no $USERNAME@${env.prod_id} \"docker pull nachocardona/train-schedule:${env.BUILD_NUMBER}\" "
                        try{
                            echo "sshpass -p '$PASSWORD' -v ssh -o StrictHostKeyChecking=no $USERNAME@${env.prod_id} \"docker stop nachocardona/train-schedule\""
                            sh "sshpass -p '$PASSWORD' -v ssh -o StrictHostKeyChecking=no $USERNAME@${env.prod_id} \"docker stop nachocardona/train-schedule\""
                            echo "sshpass -p '$PASSWORD' -v ssh -o StrictHostKeyChecking=no $USERNAME@${env.prod_id} \"docker rm nachocardona/train-schedule\""    
                            sh "sshpass -p '$PASSWORD' -v ssh -o StrictHostKeyChecking=no $USERNAME@${env.prod_id} \"docker rm nachocardona/train-schedule\""                 
                        }catch(err){
                            echo 'caught error: $err' 
                        }
                        sh "sshpass -p '$PASSWORD' -v ssh -o StrictHostKeyChecking=no $USERNAME@${env.prod_id} \"docker run -p 8080 --name  nachocardona/train-schedule -d nachocardona/train-schedule:${env.BUILD_NUMBER} \""   
                    }
                }
            }
        }
    }
}
