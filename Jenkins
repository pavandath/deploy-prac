    pipeline{
        agent any
        environment{
            SONAR_URL = "http://104.154.180.122:9000"
            SONAR_TOKEN = credentials('sonar_creds')
        }
        stages{
            stage ('Build'){
                steps{
                echo "************************RUNNING BUILD STAGE************************"
                sh 'rm -rf spring-petclinic || true'
                sh 'git clone https://github.com/pavandath/spring-petclinic.git'
                dir('spring-petclinic'){
                    sh 'mvn clean package -DskipTests -Dcyclonedx.skip=true'
                    stash name: 'zip-jar', includes: 'target/*.jar'
                }
                }
            }
            stage ('CodeQuality'){
                
                steps{
                    echo "************************RUNNING CODEQUALITY STAGE************************"
                    dir('spring-petclinic'){
                    sh '''
                    mvn clean verify sonar:sonar \
                        -Dsonar.projectKey=deploy \
                        -Dsonar.host.url=${SONAR_URL} \
                        -Dsonar.login=${SONAR_TOKEN} \
                        -DskipTests\
                        -Dcyclonedx.skip=true 
                    '''
                    }
                }
            }
            stage ('DockerBuild'){
                environment{
                    DOCKER_CREDS = credentials('docker_credens')
                }
                steps{
                echo "************************RUNNING DockerBuild STAGE************************"
                sh "mkdir build"
                dir ('build'){
                    unstash 'zip-jar'
                }
                writeFile file: 'Dockerfile',
                text: '''
                FROM eclipse-temurin:21-jdk-jammy
                WORKDIR /app
                ADD build/target/*.jar  spring.jar
                EXPOSE 8080
                CMD ["java", "-jar", "spring.jar"]
                '''
                sh "docker rmi -f pavandath510/java-spring:v1 || true"
                sh "docker build -t pavandath510/java-spring:v1 ."
                sh "docker login -u ${DOCKER_CREDS_USR} -p ${DOCKER_CREDS_PSW}"
                sh "docker push pavandath510/java-spring:v1"

            }
            post{
                always{
                    sh "docker image prune -f'"
                }
            }
            }
            stage('Deploy'){
                sh "docker rm -f $(docker ps -aq) || true"
                sh "docker run -d --name deployment -p 8080:8080 pavandath510/java-spring:v1 "
                
            }
        }
    }
