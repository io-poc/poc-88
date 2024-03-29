def isSASTEnabled
def isSASTPlusMEnabled
def isSCAEnabled
def isDASTEnabled
def isDASTPlusMEnabled


pipeline {
    agent any
    stages {
        stage('Checkout Source Code') {
            steps {
                git branch: 'master', url: 'https://github.com/io-poc/poc-88'
            }
        }

        stage('Build Source Code') {
            steps {
                  withMaven(maven: 'Maven3') {
                      sh '''mvn clean package -Dmaven.test.skip'''
                  }
            }
        } 

        stage('IO - Prescription') {
            steps {
                synopsysIO(connectors: [
                    io(
                        configName: 'io-azure',
                        projectName: 'insecure-bank',
                        workflowVersion: '2022.4.1'),
                    github(
                        branch: 'master',
                        configName: 'github-demo',
                        owner: 'io-poc',
                        repositoryName: 'poc-88'), 
                    jira(
                         assignee: 'karn@synopsys.com', 
                         configName: 'jira-sandbox', 
                         issueQuery: 'resolution=Unresolved', 
                         projectKey: 'INSEC', 
                         projectName: 'insecure-bank') 
                   /* blackduck(
                        configName: 'BIZDevBD', 
                        projectName: 'codedx_insecure', 
                        projectVersion: '1.0')        */                
                    ]) {
                        sh 'io --stage io Persona.Type=developer Project.Release.Type=major'
                    }

                script {
                    def prescriptionJSON = readJSON file: 'io_state.json'

                    isSASTEnabled = prescriptionJSON.data.prescription.security.activities.sast.enabled
                    isSASTPlusMEnabled = prescriptionJSON.data.prescription.security.activities.sastPlusM.enabled
                    isSCAEnabled = prescriptionJSON.data.prescription.security.activities.sca.enabled
                    isDASTEnabled = prescriptionJSON.data.prescription.security.activities.dast.enabled
                    isDASTPlusMEnabled = prescriptionJSON.data.prescription.security.activities.dastPlusM.enabled
                    isImageScanEnabled = prescriptionJSON.data.prescription.security.activities.imageScan.enabled

                }
            }
        }


        stage('SAST - Polaris') {
            when {
                expression { isSASTEnabled }
            }
            steps {
                echo 'Running SAST using Polaris'
                synopsysIO(connectors: [
                    [$class: 'PolarisPipelineConfig',
                    configName: 'csprod-polaris',
                    projectName: 'codedx-insecure-bank']]) {
                    sh 'io --stage execution --state io_state.json'
                }
            }
        }
        
      /*  stage('SAST- RapidScan') { environment {
            OSTYPE='linux-gnu' }
            when {
               expression { isSASTEnabled }
            }
            steps {
                echo 'Running SAST using Sigma - Rapid Scan'
                echo env.OSTYPE
                synopsysIO(connectors: [rapidScan(configName: 'sigma-sandbox')]) {
                sh 'io --stage execution --state io_state.json' }
            }
        } 
        
        stage('SAST Plus Manual') {
            when {
                expression { isSASTPlusMEnabled }
            }
            steps {
                script {
                    input message: 'Manual source code review (SAST - Manual) triggered by IO. Proceed?'
                }
                echo "Out-of-Band Activity - SAST Plus Manual triggered & approved"
            }
        } */

        stage('SCA - BlackDuck') {
            when {
                expression { isSCAEnabled }
            }
            steps {
              echo 'Running SCA using BlackDuck'
              synopsysIO(connectors: [
                  blackduck(configName: 'BIZDevBD',
                  projectName: 'codedx-insecure-bank',
                  projectVersion: '1.0')]) {
                  sh 'io --stage execution --state io_state.json'
              }
            }
        } 

       /* stage('DAST Plus Manual') {
            when {
                expression { isDASTPlusMEnabled }
            }
            steps {
                script {
                    input message: 'Manual threat-modeling (DAST - Manual) triggered by IO. Proceed?'
                }
                echo "Out-of-Band Activity - DAST Plus Manual triggered & approved"
            }
        } 

        stage('IO - Workflow') {
            steps {
                echo 'Execute Workflow Stage'
                synopsysIO(connectors: [
                    codeDx(configName: 'SIG-CodeDx', projectId: '20'),
                    jira(assignee: 'karn@synopsys.com', configName: 'jira-sandbox', issueQuery: 'resolution=Unresolved AND labels in (Security, Defect)', projectKey: 'INSEC'), 
                    //msteams(configName: 'poc-msteams'), 
                    //buildBreaker(configName: 'poc-bb')
                ]) {
                    sh 'io --stage workflow --state io_state.json'
                }
                
                 script {
                    def workflowJSON = readJSON file: 'wf-output.json'
                    print("========================== IO WorkflowEngine Summary ============================")
                    print("Breaker Status: $workflowJSON.breaker.status")
                } 
            }
        }
        
        stage('Security Sign-Off') {
            steps {
                script {

                    def workflowJSON = readJSON file: 'wf-output.json'
                    
                    
                    //Build Breaker
                    if(workflowJSON.breaker.status==true) {
                          echo "Sending Notifications to Teams..."
                          //sh '''curl -H 'Content-Type: application/json' -d '{"text": "Breaking the build for application: Insecure Bank"}' <WEBHOOK>'''
                          echo "Breaking the build based on the identified Vulnerabilities. Setting pipeline to fail"
                          //exit 1
                    }
                    
                    codedx_value = workflowJSON.summary.risk_score
                    for(arr in codedx_value){
                        if(arr != null)
                        {   
                            print("Code Dx Score: $arr")
                            if(arr < 80)
                            {
                                input message: 'Code Dx Score did not meet the defined threshold. Do you wish to proceed?'
                            }
                        }
                    }
                    
                }
                echo "Security Sign-Off triggered & approved"
            }
        } */
    }

}
