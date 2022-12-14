/* Auteur: Jospin Anogo  email: anogoj@gmail.com 
prérequis: 
1) configurer une variable "DOCKER_PASSWORD" dans les paramêtres du serveur jenkins
2) configurer les "notifications par email" sur le serveur jenkins, mois j'ai utilisé "gmail.com"
3) installer "trivy" sur le serveur jenkins 
*/
def PROJET = "Test House innovation"
def DOCKER_ID = "anogo"
def USER_EMAIL = "anogoj@gmail.com"
def IMAGE_NAME = "house-innovation"
def IMAGE_TAG_DEV = DOCKER_ID+"/"+IMAGE_NAME+"-dev:$BUILD_ID"
def IMAGE_TAG_PROD = DOCKER_ID+"/"+IMAGE_NAME+"-prod:$BUILD_ID"
def REPOSITORY = "https://github.com/Tajjospin/house_innovation.git"
def PortApp = 80
def PortContainer = 8200

pipeline{
    agent any
  stages{

    /*stage('Clone Repository'){
        steps{
            git (branch: 'main', credentialsId: credential, url: repository)
        }
    }*/

    stage('Build Docker Image'){
        steps{
            script{
                sh "docker build -t ${IMAGE_TAG_DEV} ."
            }
        }
    }

    stage('Test acceptance'){
        steps{
            script{
                
                sh "docker run --name ${IMAGE_NAME}${BUILD_ID} -d -p ${PortContainer}:${PortApp} ${IMAGE_TAG_DEV}"
                sh "curl localhost:${PortContainer} | grep -q 'Author: Roody95'"
                sh "docker stop ${IMAGE_NAME}${BUILD_ID}"
                sh "docker rm ${IMAGE_NAME}${BUILD_ID}"

            }
        }
    }

    stage('scan vulnerabilte image'){
        steps{
            script{
                sh "trivy image --no-progress --exit-code 1 --severity HIGH ${IMAGE_TAG_DEV}"
            }
        }
    }
    stage('Push DEV sur dockerhub'){
        when {
            expression {GIT_BRANCH =="origin/develop"}
        }
        steps{
            script{
              
                sh "docker login -u ${DOCKER_ID} -p ${DOCKER_PASSWORD}"
                sh "docker push ${IMAGE_TAG_DEV}"
                 
            }
        }
    }
    stage('Push Prod sur dockerhub'){
        when {
            expression {GIT_BRANCH =="origin/main"}
        } 
        steps{
            script{
                sh "docker login -u ${DOCKER_ID} -p ${DOCKER_PASSWORD}"
                sh "docker tag ${IMAGE_TAG_DEV} ${IMAGE_TAG_PROD}"
                sh "docker push ${IMAGE_TAG_PROD}"
            }
        }
    }

    stage('supression des images en local'){
          steps{
            script{
              sh '''docker images |grep "'''+DOCKER_ID+'''" | awk '{ print $3 }' | xargs --no-run-if-empty docker rmi -f'''
              sh '''docker images | awk '/^<none>/ { print $3 }' | xargs --no-run-if-empty docker rmi -f'''
              sh 'echo "image suprimé"'
            }
          }
        }


}
    post {
        failure {
            mail to: "${USER_EMAIL}",
            subject: "ECHEC DU JOB, BUILD-${BUILD_ID} : ${currentBuild.currentResult}: ${env.JOB_NAME}",
            body: "échec du job :${currentBuild.currentResult}, le projet : ${PROJET},\n Comit par: ${env.GIT_AUTHOR},message commit : ${env.GIT_COMMIT_MSG}, Branche : ${env.JOB_NAME} / ${GIT_BRANCH}"
        }
    }
}