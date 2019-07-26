podTemplate(label: 'mypod', containers: [
    containerTemplate(name: 'docker', image: 'docker', command: 'cat', ttyEnabled: true),
    containerTemplate(name: 'maven', image: 'maven', command: 'cat', ttyEnabled: true)
  ],
  volumes: [
    hostPathVolume(hostPath: '/var/run/docker.sock', mountPath: '/var/run/docker.sock'),
  ]) {
    node('mypod') {
        stage('Get Source'){
          sleep 20
          git url: 'https://github.com/dFurman/course.git'
        }

        stage('Build && SonarQube analysis') {
                withSonarQubeEnv('sonarScanner') {
                    container('maven') {
                        sh 'mvn clean package sonar:sonar'
                    }
                }
        }

        stage('Upload war file'){
          nexusArtifactUploader(
                            nexusVersion: 'nexus3',
                            protocol: 'http',
                            nexusUrl: 'nexus:8081',
                            version: '0.3.1',
                            groupId: 'time-tracker-web',
                            repository: 'maven-private',
                            credentialsId: 'nexus-creds',
                            artifacts: [
                                [artifactId: 'time-tracker-web',
                                file: 'web/target/time-tracker-web-0.3.1.war',
                                type: 'war']
                            ]
                        );
        }
        
        stage('Build && Upload Docker image'){
          container('docker'){
            withDockerRegistry(credentialsId: 'nexus-creds', url: 'http://localhost:32001') {
              sh 'docker build -t localhost:32001/time-tracker-web:0.3.1 .'
              sh 'docker push localhost:32001/time-tracker-web:0.3.1'
            }
          }
        }

    }
}