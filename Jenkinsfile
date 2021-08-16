pipeline {
    agent {
        node { label "maven" }
    }

    environment { QUAY = credentials('QUAY_USER') }


    stages {
        stage("Test") {
            steps {
                sh "./mvnw verify"
            }
        }
        stage("Build & Push Image") {
            steps {
                sh '''
                    ./mvnw quarkus:add-extension \
                    -Dextensions="container-image-jib"
                '''
                sh '''
                    ./mvnw package -DskipTests \
                    -Dquarkus.container-image.build=true \
                    -Dquarkus.container-image.registry=quay.io \
                    -Dquarkus.container-image.group=$QUAY_USR \
                    -Dquarkus.container-image.name=aro-workshop-lab \
                    -Dquarkus.container-image.username=$QUAY_USR \
                    -Dquarkus.container-image.password="$QUAY_PSW" \
                    -Dquarkus.container-image.tag=build-${BUILD_NUMBER} \
                    -Dquarkus.container-image.additional-tags=latest \
                    -Dquarkus.container-image.push=true
                '''

            }
        }
        stage('Deploy to TEST') {
            when { not { branch "main" } }

            steps {
                sh """
                    oc set image deployment home-automation \
                    home-automation=quay.io/${QUAY_USR}/aro-workshop-lab:build-${BUILD_NUMBER} \
                     -n aro-lab-test --record
                """
            }
        } 
        stage('Deploy to PROD') {
            when {  branch "main"  }

            steps {
                sh """
                    oc set image deployment home-automation \
                    home-automation=quay.io/${QUAY_USR}/aro-workshop-lab:build-${BUILD_NUMBER} \
                     -n aro-lab-prod --record
                """
            }
        } 

    }
}

