def myEchoMail(jobName,jobId,testRes){
    
    echo(message:"$jobName -- ${jobId} -- $testRes")
}

//def env.testResVar

pipeline{
    agent any
    
    parameters {
            booleanParam 'RUN_TEST'
            booleanParam 'SEND_EMAIL'
            string 'GIT_BRANCH'
    }

    stages {
        
        stage("Download"){
            
            steps{
                echo (message: "Download stage")
                cleanWs()
                dir('pipelineFolder'){
                      git(
                    branch:"${params.GIT_BRANCH}",
                    url:'https://github.com/KLevon/jenkins-course'
                    )
                }
            
            rtDownload(
                serverId:'jfrogartif',
                spec:'''{
                     "files": [
                            {
                              "pattern":"example-repo-local/printer2.zip",
                              "target": "PRINTER.zip"
                            }
                          ]
                    
                    
                }'''
                
                
                
                
                )
                
            unzip(
                zipFile:"PRINTER.zip",
                dir:"pipelineFolder"
                )
            
            
            }
        }
        
        stage("Build"){
         
            steps{
                    withCredentials (
                    [usernamePassword(credentialsId: 'kurs', passwordVariable: 'psw', usernameVariable: 'usr')]
                ){
                    echo psw
                    echo usr
                }
                            echo (message: "Build stage")
                            bat(
                                
                                script:"""
                                    
                                   cd pipelineFolder 
                                   Makefile.bat
                                    
                                """
                                )
            }
                        
               
        }
            
            
            stage("Test"){
                when{
                    equals expected: true,
                    actual: params.RUN_TEST
                }
                steps{
                    script{
                        def array=["printer","scanner","main"]
                        env.testResVar=""
                        for( element in array) {
                         env.testResVar  +=    bat(
                                    script:"""
                        
                                         cd pipelineFolder
                                         Tests.bat $element
                                           
                                    
                                        """,
                                    returnStdout:true
                            
                                    
                                ).trim()
                        }
                      
                    
                }
                }
                
                
                
            }
        
           stage("Publish"){
            
            steps{
                echo (message: "Publish stage")
                script{
                    
                    zip(
                    zipFile: "pipeline.zip",
                    archive: true,
                    dir:"pipelineFolder"  
                    )
                    
                }
                
                rtUpload(
                    serverId:'jfrogartif',
                    spec:"""{
                     "files": [
                            {
                              "pattern":"pipeline.zip",
                              "target":"example-repo-local/release/${env.BUILD_ID}/"
                            }
                          ]
                    
                    }"""
                    
                    
                    
                    
                )
                
                
            }
        }
        
    }
    
    post {
        
        success{
            
            script{
                if(params.SEND_EMAIL){
                    myEchoMail(env.JOB_NAME,"${env.BUILD_ID}",env.testResVar)
                }
            }
            
            
        }
        
        
        
    }
    
    
    
}