podTemplate(label: 'mypod', containers: [
    containerTemplate(name: 'git', image: 'alpine/git', ttyEnabled: true, command: 'cat'),
    containerTemplate(name: 'maven', image: 'maven:3.3.9-jdk-8-alpine', command: 'cat', ttyEnabled: true),
    containerTemplate(name: 'docker', image: 'docker', command: 'cat', ttyEnabled: true)
  ],
  volumes: [
    hostPathVolume(mountPath: '/var/run/docker.sock', hostPath: '/var/run/docker.sock'),
  ]
  ) {

    node('mypod') {
        stage('Check running containers') {
            container('docker') {
                // example to show you can run docker commands when you mount the socket
                sh 'hostname'
                sh 'hostname -i'
                sh 'docker ps'
            }
        }
     stage('Clone repository') {
            container('git') {
                sh 'whoami'
                sh 'hostname -i'
                sh 'git clone -b master  https://github.com/abraaojs/kubernetes-pipeline-app.git'

                    // Pega o commit id para ser usado de tag (versionamento) na imagem
                sh "git rev-parse --short HEAD > commit-id"
                tag = readFile('commit-id').replace("\n", "").replace("\r", "")
    
                // configura o nome da aplicação, o endereço do repositório e o nome da imagem com a versão
                appName = "app"
                registryHost = "127.0.0.1:30400/"
                imageName = "${registryHost}${appName}:${tag}"
            }
        }

     stage('Build') {
                sh 'hostname'
                sh 'hostname -i'
                def customImage = docker.build("${imageName}")

     }
      
    stage('Push') {
        customImage.push()
    }

    stage ('Deploy PROD') {
        input "Deploy to PROD?"
        customImage.push('latest')
        sh "kubectl apply -f https://raw.githubusercontent.com/cirolini/Docker-Flask-uWSGI/master/k8s_app.yaml"
        sh "kubectl set image deployment app app=${imageName} --record"
        sh "kubectl rollout status deployment/app"
    }

}
