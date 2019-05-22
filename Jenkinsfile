import java.text.SimpleDateFormat
//@Library('liberary')
def props
def label = "jenkins-slave-${UUID.randomUUID().toString()}"
currentBuild.displayName = new SimpleDateFormat("yy.MM.dd").format(new Date()) + "-" + env.BUILD_NUMBER

podTemplate(
  label: label,
  namespace: "java-build", // Not allowed with declarative
  serviceAccount: "build",
  yaml: """
apiVersion: v1
kind: Pod
spec:
  containers:
  - name: helm
    image: vfarcic/helm:2.9.1
    command: ["cat"]
    tty: true
    volumeMounts:
    - name: build-config
      mountPath: /etc/config
  - name: kubectl
    image: vfarcic/kubectl
    command: ["cat"]
    tty: true
  - name: golang
    image: golang:1.9
    command: ["cat"]
    tty: true
  volumes:
  - name: build-config
    configMap:
      name: build-config
"""
) {
  node(label) {
    //stage("build") {
     // container("helm") {
    //    sh "cp /etc/config/build-config.properties ."
  //      props = readProperties interpolate: true, file: "build-config.properties"
      //}
      node("docker") { // Not allowed with declarative
        checkout scm
         sh 'echo k8sBuildImageBeta(props.image)'
      }
    }
    stage("func-test") {
      try {
        container("docker") {
          checkout scm
          sh 'echo k8sUpgradeBeta(props.project, props.domain, "--set replicaCount=2 --set dbReplicaCount=1")'
        }
        container("kubectl") {
           sh 'echo k8sRolloutBeta(props.project)'
        }
        container("golang") {
           sh 'echo k8sFuncTestGolang(props.project, props.domain)'
        }
      } catch(e) {
          error "Failed functional tests"
      } finally {
        container("docker") {
           sh 'echo k8sDeleteBeta(props.project)'
        }
      }
    }
    if ("${BRANCH_NAME}" == "master") {
      stage("release") {
        node("docker") {
           sh 'echo k8sPushImage(props.image)'
        }
        container("docker") {
            sh 'echo k8sPushHelm(props.project, props.chartVer, props.cmAddr)'
        }
      }
      stage("deploy") {
        try {
          container("docker") {
            sh 'echo k8sRollback(props.project)'
          }
          container("kubectl") {
            sh 'echo k8sRollback(props.project)'
          }
          container("golang") {
            sh 'echo k8sRollback(props.project)'
          }
        } catch(e) {
          container("docker") {
            sh 'echo k8sRollback(props.project)'
          }
        }
      }
    }
  }
}
