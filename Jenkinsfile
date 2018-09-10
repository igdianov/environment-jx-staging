pipeline {
  options {
    disableConcurrentBuilds()
  }
  agent {
    // The following configuration enables scheduling builder pods on cheap GKE preemptible nodes. 
    // The kubernetes cluster must have a nodel pool with "Pre-emptible nodes” options enabled
    // GKE will add a label to each node “cloud.google.com/gke-preemptible”, so we can use it in our builder pod setup.
    // We must also add NO_SCHEDULE: gke-preemtible=true taint in order to avoid scheduling other pods on nodes in this pool
    kubernetes {
        // Change from jenkins-maven label to be able to use our yaml configuration snippet
        label "jenkins-x-maven"
        // and inherit from Jx Maven pod template
        inheritFrom "maven"
        // Add the following configuration to Jenkins builder pod template
        yaml """
spec:
  # The node selector says that pod must be scheduled only on nodes that have specific label and value. 
  # Given the nature of preemptible VMs, the instance might not be available when it’s needed. 
  nodeSelector:
    cloud.google.com/gke-preemptible: "true"

  # Instead of nodeSelector, Node affinity feature allows us to set up rules that kubernetes will follow 
  # during scheduling or execution of pods.
  # The affinity rule is saying that when kubernetes scheduling a new pod then it should prefer preemptible instance. 
  # Though if there is no available preemptible instance then it will be scheduled on a standard one. 
  #affinity:
  #  nodeAffinity:
  #    preferredDuringSchedulingIgnoredDuringExecution:
  #    - weight: 1
  #      preference:
  #        matchExpressions:
  #        - key: cloud.google.com/gke-preemptible
  #          operator: In
  #          values:
  #          - "true"
            
  # Also, add toleration to GKE preemtible pool taint to the pod in order to run it on that node pool
  tolerations:
  - key: "gke-preemptible"
    operator: "Equal"
    value: "true"
    effect: "NoSchedule"        
"""        
    }
  }
  environment {
    DEPLOY_NAMESPACE = "jx-staging"
  }
  stages {
    stage('Validate Environment') {
      steps {
        container('maven') {
          dir('env') {
            sh 'jx step helm build'
          }
        }
      }
    }
    stage('Update Environment') {
      when {
        branch 'master'
      }
      steps {
        container('maven') {
          dir('env') {
            sh 'jx step helm apply'
          }
        }
      }
    }
  }
}
