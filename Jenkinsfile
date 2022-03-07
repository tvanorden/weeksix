Skip to content
Search or jump to…
Pull requests
Issues
Marketplace
Explore
 
@tvanorden 
tvanorden
/
week6
Public
forked from Yaash19/week6
Code
Pull requests
Actions
Projects
Wiki
Security
Insights
Settings
week6/Jenkinsfile
@tvanorden
tvanorden Update Jenkinsfile
Latest commit 62fc685 5 minutes ago
 History
 2 contributors
@Yaash19@tvanorden
107 lines (106 sloc)  2.92 KB
   
podTemplate(yaml: '''
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
''') {
  node(POD_LABEL) {
    stage('Build a gradle project') {      
      container('gradle') {
        git 'https://github.com/tvanorden/week6.git'
        stage("Echo branch") {
            sh """
            echo ${env.GIT_BRANCH}
            """
        }       
        stage("Compile Java") {
            if(env.BRANCH_NAME == 'master' || env.BRANCH_NAME == 'feature' ) {
                sh "chmod +x gradlew"
                sh "./gradlew compileJava"
            }
        }  
        stage("Unit test") {
            if(env.BRANCH_NAME == 'master' || env.BRANCH_NAME == 'feature' ) {
                sh "chmod +x gradlew"
                sh "./gradlew test"
            }
        }
        stage("Code coverage") {
            if(env.BRANCH_NAME == 'master') {
                sh "chmod +x gradlew"
                sh "./gradlew jacocoTestReport"
                sh "./gradlew jacocoTestCoverageVerification"
            }
        } 
        stage("Static code analysis") {
            if(env.BRANCH_NAME == 'master' || env.BRANCH_NAME == 'feature' ) {
                sh "chmod +x gradlew"
                sh "./gradlew checkstyleMain"
            }
        }  
        
        stage('Package') {
            sh '''
            chmod +x gradlew
            ./gradlew build
            mv ./build/libs/calculator-0.0.1-SNAPSHOT.jar /mnt
            '''
            }
      }
    }
    stage('Build Java Image') {
      container('kaniko') {
        stage('Build Image') {
		 if(env.BRANCH_NAME == 'master') {
			  sh '''
			  echo 'FROM openjdk:8-jre' > Dockerfile
			  echo 'COPY ./calculator-0.0.1-SNAPSHOT.jar app.jar' >> Dockerfile
			  echo 'ENTRYPOINT ["java", "-jar", "app.jar"]' >> Dockerfile
			  mv /mnt/calculator-0.0.1-SNAPSHOT.jar .
			  /kaniko/executor --context `pwd` --destination tvan8310/calculator:1.0
			  '''
            }
		if(env.BRANCH_NAME == 'feature') {
			  sh '''
			  echo 'FROM openjdk:8-jre' > Dockerfile
			  echo 'COPY ./calculator-0.0.1-SNAPSHOT.jar app.jar' >> Dockerfile
			  echo 'ENTRYPOINT ["java", "-jar", "app.jar"]' >> Dockerfile
			  mv /mnt/calculator-0.0.1-SNAPSHOT.jar .
			  /kaniko/executor --context `pwd` --destination tvan8310/calculator-feature:0.1
			  '''
            }
        }
      }
    }
  }
}
© 2022 GitHub, Inc.
Terms
Privacy
Security
Status
Docs
Contact GitHub
Pricing
API
Training
Blog
About
Loading complete
