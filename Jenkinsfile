def JOB_NAME = "${env.JOB_BASE_NAME}"
def BUILD_NUMBER = "${env.BUILD_NUMBER}"
def POD_NAME = "jenkins-${JOB_NAME}-${BUILD_NUMBER}"
def K8S_CLUSTER = params.k8s_cluster ?: kubernetes

// Kubernetes pod template to run.
podTemplate(
    cloud: K8S_CLUSTER,
    namespace: "default",
    name: POD_NAME,
    label: POD_NAME,
    yaml: """
apiVersion: v1
kind: Pod
spec:
  containers:
  - name: runner
    image: golang:1.17-buster
    imagePullPolicy: IfNotPresent
    tty: true
    env:
    - name: DOCKER_HOST
      valueFrom:
        fieldRef:
          fieldPath: status.hostIP
    - name: DOCKER_TLS_VERIFY
      value: ""
  - name: jnlp
    args: ["\$(JENKINS_SECRET)", "\$(JENKINS_NAME)"]
    image: "jenkins/inbound-agent:4.11.2-4-alpine"
    imagePullPolicy: IfNotPresent
""",
) {
    node(POD_NAME) {
        container("runner") {
            stage("Checkout") {
                retry(10) {
                    checkout([
                        $class: 'GitSCM',
                        branches: scm.branches,
                        doGenerateSubmoduleConfigurations: scm.doGenerateSubmoduleConfigurations,
                        extensions: [[$class: 'CloneOption', noTags: false, shallow: false, depth: 0, reference: '']],
                        userRemoteConfigs: scm.userRemoteConfigs,
                    ])
                }
            }
            stage("Lint") {
                sh """
                make lint
                """
            }
            stage("Build") {
                sh """
                make docker-build
                """
            }
        }
    }
}
