pipeline {
    agent any
    stages {
        stage('checkout core') {
            steps {
                checkout([$class: 'GitSCM', branches: [[name: 'refs/tags/jenkins-2.303.3']], doGenerateSubmoduleConfigurations: false, extensions: [], submoduleCfg: [], userRemoteConfigs: [[credentialsId: 'gh-token', url: 'https://github.com/jenkinsci/jenkins.git']]])
            }
        }
        stage('parse dependencies') {
            steps {
                // sh 'mvn dependency:tree > dependencyTree.txt'
                script {
                    def dependencies = readFile 'dependencyTree.txt'
                    writeFile file: 'dep.csv', text: parseDependencies(dependencies)
                    sh 'cat dep.csv'
                }
            }
        }
    }
    
    post {
        always {
            archiveArtifacts artifacts: 'dep.csv', fingerprint: true, followSymlinks: false, onlyIfSuccessful: true
            cleanWs()
        }
    }
}

@NonCPS
def parseDependencies(dependencies) {
    def sb = new StringBuilder()
    dependencies.trim().eachLine {
        line -> sb.append(parseLine(line))
    }
    return sb.toString()
}

@NonCPS
def parseLine(line) {
    def component = (line =~ /-+< org.jenkins-ci.main:(\S+)/)
    def dependency = (line =~ /[\+\\]- (\S+):(\S+):\S+:(\S+):(\S+)(?: \((optional)\))?/)
    if (component.find()) {
        println component.group(1)
    } else if (dependency.find()) {
        def row = sprintf("%s,%s,%s,%s,%s", dependency.group(1)
                                                , dependency.group(2)
                                                , dependency.group(3)
                                                , dependency.group(4)
                                                , dependency.group(5) ?: "" )
        return row
    }
}
