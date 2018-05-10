pipeline{
    agent { label 'Docker_Slave' }
        stages{
            stage('Gitlab Checkout - Petclinic'){
            steps{
                dir('petclinic'){
                sh 'echo -e "\033[0;34m ## Gitlab Checkout ##\033[0m"'
                 git branch: '${gitlabSourceBranch}',
                 credentialsId: '76a11c66-201d-41e2-afcb-699ad2e7a7fc',
                 url: 'http://10.200.5.162/Petclinic/petclinic.git'
                }
            }
        }
            stage('Build and Analisys - Sonarqube'){
                steps{
                    dir('petclinic'){
                        sh 'echo -e "\033[0;34m ## Build and Analisys ##\033[0m"'
                        withSonarQubeEnv("Sonarqube"){
                            withMaven(globalMavenSettingsConfig: 'ea831855-652a-458a-ad8e-af374982ee01', maven: 'maven', mavenLocalRepo: '.repository') {
                                sh '''
                                    mvn clean jacoco:prepare-agent test package\
                                    jacoco:report\
                                    sonar:sonar
                                    '''
                                }
                           }
                        }
                    }
                }

            stage("Quality Gate") {
            steps{
                withSonarQubeEnv("Sonarqube") {
                sh 'echo -e "\033[0;34m ## Executando quality gate ##\033[0m"'
                // Executa o quality gate no projeto, caso não passe, o build é finalizado
                script{
                    timeout(time: 1, unit: 'HOURS') {
                        def qg = waitForQualityGate()
                        if (qg.status != 'OK') {
                            error "Pipeline aborted due to quality gate failure: ${qg.status}"
                            }
                        }
                    }
                }
            }
        }
                stage("Create Artifact"){
                    steps{
                        dir('petclinic'){
                            archiveArtifacts 'target/*.war'
                        }
                    }
                }
                stage('Deploy To Artifactory') {
                    steps{
                        sh 'echo -e "\033[0;34m ## Iniciando deploy da API no Artifactory ##\033[0m"'
                        script {
                        mavenBuilder = Artifactory.newMavenBuild()
                        buildInfo = Artifactory.newBuildInfo()
                        artifactoryServer = Artifactory.server 'artifactory'
                        // Deploy do artefato e do build info no artifactory
                        mavenBuilder.tool = 'maven'
                        mavenBuilder.deployer releaseRepo: 'petclinic', snapshotRepo: 'petclinic', server: artifactoryServer
                        mavenBuilder.deployer.deployArtifacts = true // Enable artifacts deployment during Maven run
                        mavenBuilder.run pom: 'petclinic/pom.xml', goals: 'clean install -DskipTests', buildInfo: buildInfo
                        artifactoryServer.publishBuildInfo buildInfo
                        }
                    }
               }
    }
}
