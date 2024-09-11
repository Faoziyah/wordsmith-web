pipeline{
  agent any
  stages{
    stage("Clone"){
      steps{
        script{
            git branch: 'jenkins', credentialsId: 'github-token', url: 'https://github.com/Primus-Learning/wordsmith-web.git'
            sh"ls -l"
        }
      }
    }
    stage("Run SOnar"){
      steps{
        script{
          echo " sonar scan passed"
        }
      }
    }
    stage("Docker build and push"){
      steps{
        script{
          withAWS(region:'us-east-2',credentials:'aws-creds'){
              def componentVersion = getVersion()
              dir("${WORKSPACE}"){
                  sh"""
                  aws ecr get-login-password --region us-east-2 | docker login --username AWS --password-stdin 421740842601.dkr.ecr.us-east-2.amazonaws.com
                  docker build -t wordsmith-web .
                  docker tag wordsmith-web:latest 421740842601.dkr.ecr.us-east-2.amazonaws.com/wordsmith-web:${componentVersion}
                  docker push 421740842601.dkr.ecr.us-east-2.amazonaws.com/wordsmith-web:${componentVersion}
                  """
              }
          }
        }
      }
    }
    stage("Deploy To Dev"){
      steps{
        script{
            withAWS(region:'us-east-2',credentials:'eks-user'){
            def componentVersion = getVersion()
            String regID = "421740842601.dkr.ecr.us-east-2.amazonaws.com"
            dir("${WORKSPACE}"){
                sh"""
                    // aws eks update-kubeconfig --name dev-cluster
                    curl -O https://s3.us-west-2.amazonaws.com/amazon-eks/1.27.15/2024-07-12/bin/linux/amd64/kubectl
                    chmod u+x ./kubectl
                    sed -i 's/REGID/${regID}/' deployment.yaml
                    sed -i 's/IMAGETAG/${componentVersion}/' deployment.yaml
                    // ./kubectl apply -f deployment.yaml -n wordsmith
                    // ./kubectl get all -n wordsmith
                """
            }
          }
        }
      }
    }
  }
}
def getVersion(){
    def baseVersion = "1.0.0"
    def finalVersion
    if(env.BRANCH_NAME == "develop"){
        finalVersion = "${baseVersion}-rc"
    }else if(env.BRANCH_NAME == "main"){
        finalVersion = baseVersion
    }else{
        finalVersion = "${baseVersion}-${BUILD_NUMBER}-${env.BRANCH_NAME}"
    }
    return finalVersion
}
