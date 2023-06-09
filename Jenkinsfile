pipeline {
    agent any
    
    stages {
        stage('checkout') {
            steps {
                checkout scmGit(branches: [[name: '*/main']], extensions: [], userRemoteConfigs: [[credentialsId: 'jenkinstoken', url: 'https://github.com/GogirlDipika/CProgramTest.git']])
            }
        }
        
        stage('clone') {
            steps {
                git branch: 'main', credentialsId: 'jenkinstoken', url: 'https://github.com/GogirlDipika/CProgramTest.git'
            }
        }
        stage('Cppcheck') {
            steps {
                bat 'cppcheck  . --platform=win64 --inconclusive --enable=all --rule-file=cppcheck_rules.xml --xml-version=2 --xml --output-file=cppcheck.xml'
            }
        }
        stage('Quality Gate') {
            steps {
                script{
                    // Read Cppcheck XML file into a string
                    def xmlString = readFile 'cppcheck.xml'

                    // Parse XML string using XmlSlurper
                    def xml = new XmlSlurper().parseText(xmlString)
                    
                    def errors = xml.errors.error
                    def warnings = xml.warnings

                    //def errorCount = errors.size()
                    def errorCount = errors.size()
                    def warningCount = warnings.size()
                    
                    echo "The number of errors is ${errorCount}"
                    echo "The number of warnings is ${warningCount}"

                    // Access elements in the XML document
                    //def errorCount = xml.@errors.toInteger()
                    //def warningCount = xml.@warnings.toInteger()
                    echo "Editing xml"
                    
                    echo "Create a new error element"
                    def newError = xml.errors.appendNode('error', [
                        id: 'newErrorId',
                        severity: 'error',
                        msg: 'This is a new error.',
                        verbose: 'This is a new error.',
                        cwe: '563',
                        inconclusive: 'false',
                        file0: 'Sonarproject.c'
                    ])
                    
                    echo "Set the location for the new error"
                    newError.appendNode('location', [
                        file: 'Sonarproject.c',
                        line: '6',
                        column: '3'
                    ])
                    
                    echo "Write the updated xml content back to cppcheck.xml"
                    def updatedXmlContent = groovy.xml.XmlUtil.serialize(xml)
                    writeFile file: 'cppcheck.xml', text: updatedXmlContent
                    
                    echo 'New error added to cppcheck.xml'
                    
                    // Quality Gate criteria
                    def maxErrors = 3
                    def maxWarnings = 10

                    // Check if quality gate conditions are met
                    if (errorCount <= maxErrors && warningCount <= maxWarnings) {
                        echo "Quality Gate: Passed"
                    } else {
                        //error "Quality Gate: Failed - Exceeded maximum errors or warnings"
                    }
                }
            }
        }
        stage('Clang format') {
            steps {
                println "Checking suggestions for formatting..."
                bat "clang-format --style=llvm --dry-run *.c"
                println "Reformatting started..."
                bat 'clang-format -i *.c'
                println "Reformatting done."
            }
        }
    }
    post {
        always {
            publishCppcheck pattern:'cppcheck.xml'
        }
    }
}


// clang-format -i *.c
// clang-format --style=llvm --dry-run *.c
