properties([
    pipelineTriggers([githubPush()]),
])
pipeline {
    agent any
    parameters {
        extendedChoice( name: 'TagName', defaultValue: '', description: 'tag name', 
            type: 'PT_SINGLE_SELECT', 
            groovyScript: """def gettags = ("git ls-remote -t https://github.com/hqzhang/octest.git").execute()
               return gettags.text.readLines().collect { it.split()[1].replaceAll('refs/tags/', '').replaceAll("\\\\^\\\\{\\\\}", '')}
                          """,)
        choice(name: 'project', choices: ['localOC'], description: 'input cluster') 
        choice(name: 'cluster', choices: ['myproject'], description: 'input project(namespace)')
        booleanParam(name: 'deployCfg', defaultValue: true, description: 'deploy deployConfig')
        booleanParam(name: 'imgStr', defaultValue: true, description: 'deploy imageStream')
        booleanParam(name: 'service', defaultValue: true, description: 'deploy service')
        booleanParam(name: 'route', defaultValue: true, description: 'deploy route')
        booleanParam(name: 'configMaps', defaultValue: true, description: 'deploy configmaps') 
        booleanParam(name: 'secrets', defaultValue: true, description: 'deploy secrets')
        string(name: 'StringSet', defaultValue: 'Mr Jenkins', description: 'Who should I say hello to?')
        password(name: 'PasswordSet', defaultValue: 'SECRET', description: 'Enter a password')
    }
    //triggers { pollSCM('*/1 * * * *') }
    options { timestamps () }
    stages {
        stage('Sonarqube Scanner') {
           environment {
               scanner = tool 'Sonar-Scanner'
           }
           steps {
               script {
                   withSonarQubeEnv('SonarQubeServer') {
                   println(scanner);
                 //sh """${scanner}/bin/sonar-scanner -v """
                 sh """${scanner}/bin/sonar-scanner  \
                      -Dsonar.projectKey=mytest     \
                      -Dsonar.java.binaries=/Users/hongqizhang/workspace/sonartest/PetClinic/target/classes \
                  """
               }
      }}}
       stage('Maven Sonar Scanner') {
           environment {
               scanner = tool 'Sonar-Scanner'
           }
           steps {
               script {
               println(scanner);
              /*def nodeImg = docker.image('node:14.15.1-alpine3.10');
              def mvnImg = docker.image('maven:3.6.3-jdk-8');
               
               nodeImg.pull();
               mvnImg.pull();*/
               def globalvar="--batch-mode -Dmaven.repo.local=/Users/hongqizhang/.m2"
               sh """mvn -X $globalvar
                      sonar:sonar                            \
                      -Dsonar.host.url=http://localhost:9000 \
                      -Dsonar.projectName=petclinic          \
                      -Dsonar.projectVersion=$BUILD_NUMBER   \
                      -Dsonar.branch.name=master             \
                      -Dsonar.projectKey=mytest
              """
             /* mvnImg.inside("-v /AppData/jenkins:/AppData/jenkins") {
                     sh """mvn -X $globalvar 
                      sonar:sonar                            \
                      -Dsonar.host.url=http://localhost:9000 \
                      -Dsonar.projectName=petclinic          \
                      -Dsonar.projectVersion=$BUILD_NUMBER   \
                      -Dsonar.branch.name=master             \
                      -Dsonar.projectKey=mytest
                      """
               }*/
             /**
              nodeImg.inside(''){
                 sh """${scanner}/bin/sonar-scanner -v """
                 sh """npm install"""
                 sh """${scanner}/bin/sonar-scanner          \
                      -Dsonar.host.url=http://localhost:9000 \
                      -Dsonar.projectName=petclinic          \
                      -Dsonar.projectVersion=$BUILD_NUMBER   \
                      -Dsonar.branch.name=master             \
                      -Dsonar.projectKey=mytest
                  """
               }
              **/
                 
            }
          }
        }

        ////// 
    
        stage('Stage: Verification Test'){
            steps {
                script {
                    echo "Stage: Verification Test"
                  /**  openshift.withCluster(url, token){
                        openshift.withProject(proj){  
                            echo "Get route url.."
                            def route  = openshift.selector('route','mynginx').object().spec.host
                    	    echo "${route}"
                     	    def myres="Welcome"
                            echo "curl route url"
                     	    def mycurl= "curl -k https://$route | grep $myres"
                     	    def proc= mycurl.execute().text
                            echo "compare result..."
                     	    def myproc= proc.contains(myres)
                     	    println(myproc)
                     	    if ( myproc) {
                         	echo "TEST PASS!"
                                writeFile(file: 'release', text: 'sbms-release-v3')
                                archiveArtifacts artifacts: "release"
                                //mail bcc: '', body: 'just test', cc: '', from: '', replyTo: '', subject: 'test', to: 'zhanghongqi@hotmail.com'
                                sh "open https://$route"
                     	    }
                     	    else {
                         	echo "TEST ERROR!"
                                //mail bcc: '', body: 'just test', cc: '', from: '', replyTo: '', subject: 'test', to: 'zhanghongqi@hotmail.com'
                                exit 1;
                            }
                        }
                    }**/                           
                }
            }
        }
       //stage
    }
}

