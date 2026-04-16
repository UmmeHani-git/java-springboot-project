pipeline {
    agent any

    environment {
        EC2_USER_HOST = 'ec2-user@54.160.208.75'
        BACKEND_PORT  = '8084'
        FRONTEND_PORT = '8501'
        RDS_HOST      = 'database-1.cxeakuo04ry1.us-east-1.rds.amazonaws.com'
        RDS_PORT      = '3306'
        RDS_USER      = 'admin'
        JAR_NAME      = 'datastore-0.0.7.jar'
    }

    stages {

        stage('Checkout') {
            steps {
                git branch: 'main',
                    url: 'https://github.com/UmmeHani-git/java-springboot-project.git'
            }
        }

        stage('Build Backend') {
            steps {
                dir('backend') {
                    sh 'mvn clean package -DskipTests=true'
                }
            }
        }

        stage('Deploy Backend') {
            steps {
                withCredentials([
                    sshUserPrivateKey(credentialsId: 'ec2-ssh', keyFileVariable: 'SSH_KEY'),
                    string(credentialsId: 'mysql-password', variable: 'MYSQL_PASSWORD')
                ]) {
                    sh """
                        scp -i \$SSH_KEY -o StrictHostKeyChecking=no \\
                            backend/target/${JAR_NAME} \\
                            \${EC2_USER_HOST}:/home/ec2-user/
                          ssh -i $SSH_KEY -o StrictHostKeyChecking=no ${EC2_USER_HOST} '
                           sudo pkill -f "datastore-0.0.7.jar" || true
                           sleep 2

                           nohup env \
                           MYSQL_HOST=... \
                           MYSQL_PORT=3306 \
                           MYSQL_USERNAME=admin \
                           MYSQL_PASSWORD="$MYSQL_PASSWORD" \
                           java -jar /home/ec2-user/datastore-0.0.7.jar \
                          --server.port=8084 \
                            > /home/ec2-user/datastore.log 2>&1 &
                      '
                    """
                }
            }
        }

        stage('Deploy Frontend') {
            steps {
                withCredentials([
                    sshUserPrivateKey(credentialsId: 'ec2-ssh', keyFileVariable: 'SSH_KEY')
                ]) {
                    sh """
                        ssh -i \$SSH_KEY -o StrictHostKeyChecking=no \${EC2_USER_HOST} '
                            cd /home/ec2-user

                            if [ -d "Java-springboot-project" ]; then
                                cd Java-springboot-project && git pull
                            else
                                git clone https://github.com/UmmeHani-git/java-springboot-project.git
                                cd Java-springboot-project
                            fi

                            pkill -f "streamlit" || true
                            sleep 2

                            cd frontend

                            python3 -m venv venv || true
                            source venv/bin/activate

                            pip install -r requirements.txt -q

                            nohup streamlit run app.py \\
                              --server.port=${FRONTEND_PORT} \\
                              > /home/ec2-user/streamlit.log 2>&1 &
                        '
                    """
                }
            }
        }
    }

    post {
        success {
            echo "App is live!"
            echo "Frontend: http://54.160.208.75:8501"
            echo "Backend:  http://54.160.208.75:8084"
        }
        failure {
            echo "Pipeline failed. Check logs above."
        }
    }
}
