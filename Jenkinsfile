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
  - name: kubectl
    image: vfarcic/kubectl
    command: ["cat"]
    tty: true
  - name: golang
    image: golang:1.9
    command: ["cat"]
    tty: true
 """
) {
  node(label) {
    stage("build") {
      container("docker") {
        // sh "cp /etc/config/build-config.properties ."
       // props = readProperties interpolate: true, file: "build-config.properties"
      }
      node(label) { // Not allowed with declarative
       // checkout scm
         sh "echo k8sBuildImageBeta"
      }
    }
    stage("func-test") {
      try {
        container("docker") {
         // checkout scm
          sh "echo k8sUpgradeBeta"
        }
        container("kubectl") {
           sh "echo k8sRolloutBeta"
        }
        container("golang") {
           sh "echo k8sFuncTestGolang"
        }
      } catch(e) {
          error "Failed functional tests"
      } finally {
        container("docker") {
           sh "echo k8sDeleteBeta"
        }
      }
    }
    if ("${BRANCH_NAME}" == "master") {
      stage("release") {
        node(label) {
           sh "echo k8sPushImage"
        }
        container("docker") {
            sh "echo k8sPushHelm"
        }
      }
      stage("deploy") {
        try {
          container("docker") {
            sh "echo k8sRollback"
          }
          container("kubectl") {
            sh "echo k8sRollbac"
          }
          container("golang") {
            sh "echo k8sRollback"
          }
        } catch(e) {
          container("docker") {
            sh "echo k8sRollback"
          }
        }
      }
    }
  }
}
