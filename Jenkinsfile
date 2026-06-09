pipeline {

    agent any

    environment {
        IMAGE = "product-api:${BUILD_NUMBER}"
        NETWORK = "app-net"
        MYSQL_CONT = "app-mysql"
        API_CONT = "library-backend"
        MYSQL_PWD = "root"
        MYSQL_DB = "librarydb"
    }

    stages {

        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Build Docker Image') {
            steps {
                bat "docker build -t %IMAGE% ."
            }
        }

        stage('Start MySQL') {
            steps {
                bat """
                docker network create %NETWORK% 2>nul

                docker rm -f %MYSQL_CONT% 2>nul

                docker run -d --name %MYSQL_CONT% --network %NETWORK% ^
                    -e MYSQL_ROOT_PASSWORD=%MYSQL_PWD% ^
                    -e MYSQL_DATABASE=%MYSQL_DB% ^
                    -p 3306:3306 ^
                    mysql:8.0

                echo Waiting for MySQL to initialise...
                ping 127.0.0.1 -n 30 > nul                """
            }
        }

        stage('Run API') {
            steps {
                bat """
                docker rm -f %API_CONT% 2>nul

                docker run -d --name %API_CONT% --network %NETWORK% ^
                    -e ConnectionStrings__DefaultConnection=Server=%MYSQL_CONT%;Port=3306;Database=%MYSQL_DB%;User=root;Password=%MYSQL_PWD%; ^
                    -e Jwt__Key=YourSuperSecretKeyThatIsAtLeast32CharactersLong ^
                    -p 5263:8080 ^
                    %IMAGE%
                """
            }
        }
    }
}
