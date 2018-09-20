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
            command: 'cat'
            envVars: [
                envVar(key: 'TILLER_NAMESPACE', value: 'team-gold'),
                secretEnvVar(key: 'CA_CERT', secretName: 'helm-secrets', secretKey: 'ca_cert'),
                secretEnvVar(key: 'HELM_CERT', secretName: 'helm-secrets', secretKey: 'helm_cert'),
                secretEnvVar(key: 'HELM_KEY', secretName: 'helm-secrets', secretKey: 'helm_key'),
            ]
        )
    ],
    volumes: [
        hostPathVolume(
            hostPath: '/var/run/docker.sock',
            mountPath: '/var/run/docker.sock'
        )
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
                sh "/helm upgrade --tiller-namespace team-gold --tls --tls-ca-cert $CA_CERT --tls-cert $HELM_CERT --tls-key $HELM_KEY --install --wait --namespace team-gold --set image.repository=${repository},image.tag=${commitId} hello hello"
            }
        }
    }
}
