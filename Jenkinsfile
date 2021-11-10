pipeline {
    agent any
    parameters {
        string(name: 'VERSION', defaultValue: '2.303.3', description: 'Jenkins core version')
    }
    stages {
        stage('checkout core') {
            steps {
                checkout([$class: 'GitSCM', branches: [[name: "refs/tags/jenkins-${params.VERSION}"]], doGenerateSubmoduleConfigurations: false, extensions: [], submoduleCfg: [], userRemoteConfigs: [[credentialsId: 'gh-token', url: 'https://github.com/jenkinsci/jenkins.git']]])
            }
        }
        stage('parse dependencies') {
            steps {
                sh 'mvn dependency:tree > dependencyTree.txt'
                script {
                    def dependencies = readFile 'dependencyTree.txt'
                    writeFile file: 'dependencies.csv', text: parseDependencies(dependencies)
                }
            }
        }
    }
    
    post {
        always {
            archiveArtifacts artifacts: 'dependencies.csv', fingerprint: true, followSymlinks: false, onlyIfSuccessful: true
            cleanWs()
        }
    }
}

@NonCPS
def parseDependencies(dependencies) {
    def sb = new StringBuilder()
    sb.append("group id,artifact id,version,scope,optional\n")
    dependencies.eachLine {
        def result = parseLine(it)
        if (result != null) {
           sb.append(result)
        }
    }
    return sb.toString()
}

@NonCPS
def parseLine(line) {
    def component = (line =~ /-+< org.jenkins-ci.main:(\S+)/)
    def dependency = (line =~ /[\+\\]- (\S+):(\S+):\S+:(\S+):(\S+)(?: \((optional)\))?/)
    if (component.find()) {
        return "${component.group(1)}\n"
    } else if (dependency.find()) {
        def row = sprintf("%s,%s,%s,%s,%s\n", dependency.group(1)
                                            , dependency.group(2)
                                            , dependency.group(3)
                                            , dependency.group(4)
                                            , dependency.group(5) ?: "" )
        return row
    }
}
