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
        state("Deploy to Production"){
            when{
                branch 'master'
            }
            steps{
                input 'Deploy to Production?'
                milestone(1)
                withCredentials([usernamePassword(credentiaslId: 'webserver_login', usernameVariable: 'USERNAME', passwordVariable: 'PASSWORD' )]){
                    script{
                        sh "sshpass -p '$PASSWORD' -v ssh -o StrictHostKeyChecking=no $USERNAME@${env.prod_id} \"docker pull nachocardona/train-schedule:${env.BUILD_NUMBER}\" "
                        try{
                            sh "sshpass -p '$PASSWORD' -v ssh -o StrictHostKeyChecking=no $USERNAME@${env.prod_id} \"docker stop nachocardona/train-schedule\""
                            sh "sshpass -p '$PASSWORD' -v ssh -o StrictHostKeyChecking=no $USERNAME@${env.prod_id} \"docker rm nachocardona/train-schedule\""                 
                        }catch(err){
                            echo: 'caught error: $err' 
                        }
                        sh "sshpass -p '$PASSWORD' -v ssh -o StrictHostKeyChecking=no $USERNAME@${env.prod_id} \"docker run -p 8080 --name  nachocardona/train-schedule -d nachocardona/train-schedule:${env.BUILD_NUMBER} \""   
                    }
                }
            }
        }
    }
}
