def flag = false;
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
      stage('Run Test') { 
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
          }
        } 
      }
      stage('Checkstyle Test') {
        when {
          expression {
            return env.BRANCH_NAME == 'main'
            return env.BRANCH_NAME == 'feature'
          }
        }
        steps {
          container('gradle') {
          sh './gradlew checkstyleMain'
          }
        }
        post {
          success {
            // pubish html
            publishHTML (target: [
              alwaysLinkToLastBuild: true,
              reportDir: 'build/reports/checkstyle/',
              reportFiles: 'main.html',
              reportName: 'CheckStyle Report'
            ])
          }
        }
      } 
      stage('Code Coverage Test') {
        when {
          branch 'main'
        }
        steps {
          container('gradle') {
            sh './gradlew jacocoTestCoverageVerification'
            sh './gradlew jacocoTestReport'
          }
        }
        post {
          success {
            // publish report
            publishHTML (target: [
              reportDir: 'build/reports/jacoco/test/html',
              reportFiles: 'index.html',
              reportName: 'JaCoCo Coverage Report'
            ])
          }
        }
      }
      stage('build a gradle project') {
        when {
          not {
            branch 'playground'
          }
        }
        try {
          steps {
            container('gradle') {
              sh './gradlew build'
              sh 'mv ./build/libs/calculator-0.0.1-SNAPSHOT.jar /mnt'
            }
            script { flag = true }
          }
        }
      catch (Exception e) {
        unstable("[ERROR]: ${STAGE_NAME} failed!")
      }
    }
      stage('Build Release Java Image') {
        // container name is repository/image:version
        when {
          expression {
            flag == true &&
            return env.GIT_BRANCH == 'main'
          }
        }
        steps {
          container('kaniko') {
            sh '''
              echo 'FROM openjdk:8-jre' > Dockerfile
              echo 'COPY ./calculator-0.0.1-SNAPSHOT.jar app.jar' >> Dockerfile
              echo 'ENTRYPOINT ["java", "-jar", "app.jar"]' >> Dockerfile
              mv /mnt/calculator-0.0.1-SNAPSHOT.jar .
              /kaniko/executor --context $(pwd) --destination kishank007/calculator:1.0
            '''
          }
        }
      }
      stage('Build Feature Java Image') {
        when {
          branch 'feature'
        }
        steps {
          container('kaniko') {
            sh '''
              echo 'FROM openjdk:8-jre' > Dockerfile
              echo 'COPY ./calculator-0.0.1-SNAPSHOT.jar app.jar' >> Dockerfile
              echo 'ENTRYPOINT ["java", "-jar", "app.jar"]' >> Dockerfile
              mv /mnt/calculator-0.0.1-SNAPSHOT.jar .
              /kaniko/executor --context $(pwd) --destination kishank007/calculator-feature:1.0
            '''
          }
        }
      } 
    }
  }      
