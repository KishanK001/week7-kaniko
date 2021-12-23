pipeline {
  agent {
    kubernetes {
      yaml ''' 
    apiVersion: v1 
    kind: Pod 
    spec: 
      containers: 
      - name: gradle 
        image: gradle:6.3-jdk14 
        command: 
        - sleep 
        args: 
        - 99d 
        volumeMounts: 
        - name: shared-storage 
          mountPath: /mnt         
      - name: kaniko 
        image: gcr.io/kaniko-project/executor:debug 
        command: 
        - sleep 
        args: 
        - 9999999 
        volumeMounts: 
        - name: shared-storage 
          mountPath: /mnt 
        - name: kaniko-secret 
          mountPath: /kaniko/.docker 
      restartPolicy: Never 
      volumes: 
      - name: shared-storage 
        persistentVolumeClaim: 
          claimName: jenkins-pv-claim 
      - name: kaniko-secret 
        secret: 
            secretName: dockercred 
            items: 
            - key: .dockerconfigjson 
              path: config.json
 ''' }
    }
    stages {
      stage('Run pipeline') { 
        when {
          anyOf {
            branch 'main'
            branch 'feature'
          }
        }
        steps { 
        container('gradle') {
          sh 'ls -la'
          sh 'gradle wrapper'
          sh 'chmod +x ./gradlew'
          sh './gradlew test' 
#          dir('Chapter08/sample1') { 
#            sh ''' chmod +x ./gradlew
#              ./gradlew build 
#              mv ./build/libs/calculator-0.0.1-SNAPSHOT.jar /mnt 
#              ''' 
#            }
          }
        } 
      }
      stage('Build Java Image') { 
        steps {
          container('kaniko') { 
              sh ''' 
              echo 'FROM openjdk:8-jre' > Dockerfile 
              echo 'COPY ./calculator-0.0.1-SNAPSHOT.jar app.jar' >> Dockerfile 
              echo 'ENTRYPOINT ["java", "-jar", "app.jar"]' >> Dockerfile 
              mv /mnt/calculator-0.0.1-SNAPSHOT.jar . 
              ls
              /kaniko/executor --context `pwd` --destination kishank007/hello-kaniko:1.2 
              '''
          }
        }
      }
    }
  }
