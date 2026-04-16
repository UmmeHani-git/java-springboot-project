pipeline {
    agent any

    environment {
        EC2 = 'ec2-user@54.160.208.75'
        JAR = 'datastore-0.0.7.jar'
    }

    stages {

        stage('Checkout') {
            steps {
                git 'https://github.com/UmmeHani-git/java-springboot-project.git'
            }
        }

        stage('Build Backend') {
            steps {
                dir('backend') {
                    sh 'mvn clean package -DskipTests'
                }
            }
        }

        stage('Deploy Backend') {
            steps {
                withCredentials([sshUserPrivateKey(credentialsId: 'ec2-ssh', keyFileVariable: 'KEY')]) {
                    sh '''
                        scp -i $KEY -o StrictHostKeyChecking=no backend/target/$JAR $EC2:/home/ec2-user/

                        ssh -i $KEY -o StrictHostKeyChecking=no $EC2 "
                            sudo pkill -f $JAR || true
                            nohup java -jar /home/ec2-user/$JAR --server.port=8084 > app.log 2>&1 &
                        "
                    '''
                }
            }
        }

        stage('Deploy Frontend') {
            steps {
                withCredentials([sshUserPrivateKey(credentialsId: 'ec2-ssh', keyFileVariable: 'KEY')]) {
                    sh '''
                        ssh -i $KEY -o StrictHostKeyChecking=no $EC2 "
                            cd /home/ec2-user

                            if [ ! -d project ]; then
                                git clone https://github.com/UmmeHani-git/java-springboot-project.git project
                            fi

                            cd project/frontend

                            sudo pkill -f streamlit || true

                            python3 -m venv venv || true
                            source venv/bin/activate
                            pip install -r requirements.txt

                            nohup streamlit run app.py --server.port=8501 > streamlit.log 2>&1 &
                        "
                    '''
                }
            }
        }
    }

    post {
        success {
            echo "✅ App Deployed Successfully"
            echo "Frontend: http://54.160.208.75:8501"
            echo "Backend:  http://54.160.208.75:8084"
        }
        failure {
            echo "❌ Deployment Failed"
        }
    }
}
