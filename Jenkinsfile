//Jenkinsfile (Scripted Pipeline)
import groovy.json.JsonSlurper
//import java.util.Properties
//
def  myapp= "wavecloud/nginx-opens"
def  myarg=" -f Dockerfile"
def myport=8081
def url="insecure://127.0.0.1:8443"
def token=""
def proj="myproject"
def cmd=""
def deleteAll(openshift,delSct = true, delCfgmap = true) {
    def obj = 'all'
    if (delSct ){ obj += ',secret'}
    if (delCfgmap ){ obj += ',configmap'}
    return openshift.delete(getObj(delSct), "--all", "--ignore-not-found=true")
}

def createEffectParams(openshift, parameter ){
       properties = new Properties()
       def read = new StringReader(parameter)
       properties.load(read)
       def paramsMap = properties.collect{key,value-> "$key=$value"}
       println "paramsMap=$paramsMap"
    return paramsMap
}        
def deployTemplate(openshift,template, paramsMap){
   echo "enter deployTemplate()"
   def processParams = paramsMap 
   if(paramsMap) {
       processParams.add(0,"-p")
       processParams.add("--ignore-unknown-parameters=true")
   }
   println "8888processParams=$processParams"
   return openshift.create(openshift.process(template, processParams))
}

def updateTemplateFile(template,deployCfg, imgStr, service, route, cfgMaps, secrets){
   echo "enter updateTemplateFile()"
   def configs = [deployCfg, imgStr, service, route, cfgMaps, secrets]
   def kinds = ["DeploymentConfig","ImageStream","Service","Route","ConfigMap","Secret"]

   for (obj in template.objects){
      echo "kind=${obj.kind}"
      [kinds,configs].transpose().each { kind, config ->  
         if(obj.kind==kind && !config){
             println "Delete $kind"
             template.objects -= obj
         }
       }
   }
   writeYaml(file:'myfile',data: template, overwrite:true )
   def tempString = readFile('myfile')
   echo "exit updateTemplateFile"
   return tempString
}

pipeline {
    agent any
    parameters {
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
        //////
        stage('Sonarqube') {
           environment {
               scannerHome = tool 'SonarQubeScanner'
           }
           steps {
               withSonarQubeEnv('sonarqube') {
                 sh "${scannerHome}/bin/sonar-scanner"
           }
           timeout(time: 10, unit: 'MINUTES') {
                waitForQualityGate abortPipeline: true
           }
    }
}
        ////// 
    
        stage('Stage: Verification Test'){
            steps {
                script {
                    echo "Stage: Verification Test"
                    openshift.withCluster(url, token){
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
                    }                           
                }
            }
        }
       //stage
    }
}

