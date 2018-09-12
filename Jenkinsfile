podTemplate(label: 'mypod', yaml: """
    apiVersion: v1
    kind: Pod
    spec:
      hostAliases:
      - ip: "127.0.0.1"
        hostnames:
        - "dockerreg.elama.ru"
    """),

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
        ),
        containerTemplate(
            name: 'kube-registry-proxy', 
            image: 'gcr.io/google_containers/kube-registry-proxy:0.4',
            ttyEnabled: true,
            envVars: [
                envVar(key: 'REGISTRY_HOST', value: 'kube-registry.cloud.svc.cluster.local'),
                envVar(key: 'REGISTRY_PORT', value: '5000')
            ],
            ports: [portMapping(name: 'registry', containerPort: 80)],
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
                def registryIp = 'dockerreg.elama.ru'
                repository = "${registryIp}/hello"
                sh "ping -c 1 dockerreg.elama.ru"
                sh "docker build -t ${repository}:${commitId} ."
                sh "docker push ${repository}:${commitId}"
            }
        }
        stage ('Deploy') {
            container ('helm') {
                sh "/helm init --client-only --skip-refresh"
                sh "/helm upgrade --install --wait --set image.repository=${repository},image.tag=${commitId} hello hello"
            }
        }
    }
}
