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
      container("kubectl") {
        // sh "cp /etc/config/build-config.properties ."
       // props = readProperties interpolate: true, file: "build-config.properties"
      }
      node("cen") { // Not allowed with declarative
       // checkout scm
         sh "echo k8sBuildImageBeta"
         sh "whoami"
      }  sh " docker version"
    }
    stage("func-test") {
      try {
        container("kubectl") {
         // checkout scm
          sh "echo k8sUpgradeBeta"
        }
        container("kubectl") {
           sh "echo k8sRolloutBeta"
        }
        container("kubectl") {
           sh "echo k8sFuncTestGolang"
        }
      } catch(e) {
          error "Failed functional tests"
      } finally {
        container("kubectl") {
           sh "echo k8sDeleteBeta"
        }
      }
    }
    if ("${BRANCH_NAME}" == "master") {
      stage("release") {
        node(label) {
           sh "echo k8sPushImage"
        }
        container("kubectl") {
            sh "echo k8sPushHelm"
        }
      }
      stage("deploy") {
        try {
          container("kubectl") {
            sh "echo k8sRollback"
          }
          container("kubectl") {
            sh "echo k8sRollbac"
          }
          container("kubectl") {
            sh "echo k8sRollback"
          }
        } catch(e) {
          container("kubectl") {
            sh "echo k8sRollback"
          }
        }
      }
    }
  }
}
