podTemplate(label: 'mypod',serviceAccount: 'tiller',
    containers: [
        containerTemplate(
            name: 'golang',
            image: 'golang:1.10-alpine',
            ttyEnabled: true,
            command: 'cat'
        ),
        containerTemplate(
            name: 'docker',
            image: 'docker:18.02',
            ttyEnabled: true,
            command: 'cat'
        ),
        containerTemplate(
            name: 'helm',
            image: 'ibmcom/k8s-helm:v2.6.0',
            ttyEnabled: true,
            command: 'cat',
        )
    ],
    volumes: [
        hostPathVolume(
            hostPath: '/var/run/docker.sock',
            mountPath: '/var/run/docker.sock'
        ),
        secretVolume(mountPath: '/root/.helm/', secretName: 'helm-secrets')
    ]
) {
    node('mypod') {
        def commitId
        stage ('Extract') {
            checkout scm
            commitId = sh(script: 'git rev-parse --short HEAD', returnStdout: true).trim()
        }
        stage ('Build') {
            container ('golang') {
                sh 'CGO_ENABLED=0 GOOS=linux go build -a -installsuffix cgo -o main .'
            }
        }
        def repository
        stage ('Docker') {
            container ('docker') {
                def registryIp = 'localhost'
                repository = "${registryIp}:80/hello1"
                sh "docker build -t ${repository}:${commitId} ."
                sh "docker push ${repository}:${commitId}"
            }
        }
        stage ('Deploy') {
            container ('helm') {
                sh "/helm init --client-only --skip-refresh"
                sh "/helm upgrade --tiller-namespace team-lime --namespace agency-stage --tls --install --tls-ca-cert /root/.helm/ca.cert --tls-cert /root/.helm/cert.pem --tls-key /root/.helm/key.pem --wait --set image.repository=${repository},image.tag=${commitId} hello hello"
            }
        }
    }
}
