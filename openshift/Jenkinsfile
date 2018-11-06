pipeline {
  agent any
  parameters {
    string(name: 'BRANCH', defaultValue: 'puma', description: 'This is the source branch for the build')
  }
  stages {
    stage('Build') {
       when {
            expression {
                openshift.withCluster() {
                    openshift.withProject() {
                        return !openshift.selector('bc', "pingapp-${params.BRANCH}").exists()
                    }
                }
            }
      }
      steps {
        script {
          openshift.withCluster() {
              openshift.withProject() {
                echo "building Branch: ${params.BRANCH}"
                checkout([$class: 'GitSCM', branches: [[name: "${params.BRANCH}"]], doGenerateSubmoduleConfigurations: false, extensions: [], submoduleCfg: [], userRemoteConfigs: [[url: 'https://github.com/nicopaez/pingapp-config.git']]])
                openshift.create('configmap', "pingapp-${params.BRANCH}",'--from-file=config.json')
                openshift.raw('label', 'configmap',"pingapp-${params.BRANCH}","app=pingapp-${params.BRANCH}")
                openshift.newApp('openshift/ruby:2.4~https://github.com/nicopaez/pingapp.git#' + "${params.BRANCH}", "--name=pingapp-${params.BRANCH}").narrow('svc').expose()

                def dcpatch = [
                  "metadata":[
                    "name":"pingapp-${params.BRANCH}"
                  ],
                  "apiVersion":"apps.openshift.io/v1",
                  "kind":"DeploymentConfig",
                  "spec": [
                    "template":[
                      "metadata":[:],
                      "spec":[
                        "containers":[[
                            "name":"pingapp-${params.BRANCH}",
                            "image": "pingapp-${params.BRANCH}:latest",
                            "resources":[:],
                            "env":[
                              [
                                "name":"CONFIG_LOCATION",
                                "value" : "/deployments/config/config.json"
                              ]
                            ],
                            "volumeMounts":[
                              [
                                "mountPath": "/deployments/config",
                                "name": "config"
                              ]
                            ]
                          ]
                        ],
                        "securityContext":[:],
                        "volumes":[
                            [
                              "configMap":
                              [
                                "name":"pingapp-${params.BRANCH}"
                              ],
                              "name":"config"
                            ]
                        ]                        
                      ]
                    ]
                  ]
                ]
                openshift.apply(dcpatch) 
            }            
          }
        }
      }
    }
  }
}
