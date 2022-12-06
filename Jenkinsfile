pipeline {
    agent none
     parameters{
        string defaultValue: 'debian_server', description: 'Nombre del Nodo del ambiente de Desarrollo (DEV), ', name: 'LABEL_DEV_NODE', trim: false
        string defaultValue: 'container', description: 'Nombre del Nodo  del ambiente QA, ', name: 'LABEL_QA_NODE', trim: false        
        string defaultValue: 'master', description: 'Nombre del NODO del ambiente PRODUCCION, ', name: 'LABEL_PROD_NODE', trim: false
     }

    environment{
        DEV_NODE="${params.LABEL_DEV_NODE}"
        QA_NODE="${params.LABEL_QA_NODE}"
        PROD_NODE="${params.LABEL_PROD_NODE}"
    }

    stages{
        stage("Clone Repository"){
            agent { label DEV_NODE }
            steps{
                git branch: 'main', url: 'https://github.com/yovi123paye/proyecto-final-vue.git'                                
                sh "docker build -t proyecto_final_vue/final:v1 ."                
                sh "echo Repositorio Clonado en DEV!"
            }
        }
        stage("Preparando Docker image y Desplegando de DEV"){
            agent { label DEV_NODE }
            steps{                                             
                sh "docker save -o construyeya.tar proyecto_final_vue/final:v1"
                stash name: "stash-artifact", includes: "construyeya.tar"
                archiveArtifacts 'construyeya.tar'
                sh "echo Imagen creada compresa"
                sh "docker run -p 8081:80 -d --name dev_proyecto_final_vue proyecto_final_vue/final:v1"
            }
        }
        stage("Desplegando Entorno QA"){
            agent { label QA_NODE }
            steps{
                sh "echo Desplegando Entorno QA"
                unstash "stash-artifact"
                sh "docker load -i construyeya.tar"                              
                sh "docker run -p 8081:80 -d --name qa_proyecto_final_vue proyecto_final_vue/final:v1"
                sh "echo QA publicado"
                
            }
        }
        stage("Run Automation tests"){
            agent { label QA_NODE}
            steps {                
                sh "echo TEST QA"
                sh "curl http://localhost:8081"
            }

         }
        
        stage("Deploy in PROD environment"){
            agent { label PROD_NODE}
            steps {
                sh "echo Despliegue en PROD"          
                sh "docker run -p 8082:80 -d --name prod_proyecto_final_vue proyecto_final_vue/final:v1"
            }

        }
    }

}
