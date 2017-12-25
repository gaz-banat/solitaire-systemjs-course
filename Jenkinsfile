stage 'CI'

node {

	checkout scm
    // git branch: 'jenkins2-course', 
    //    url: 'https://github.com/g0t4/solitaire-systemjs-course'

    // pull dependencies from npm
    // on windows use: bat 'npm install'
    sh 'npm install'

    // stash code & dependencies to expedite subsequent testing
    // and ensure same code & dependencies are used throughout the pipeline
    // stash is a temporary archive
    //stash name: 'everything', 
    //      excludes: 'test-results/**', 
     //     includes: '**'
    
    
}


stage 'Browser Testing'
node {
    // test with PhantomJS for "fast" "generic" results
    // on windows use: bat 'npm run test-single-run -- --browsers PhantomJS'
    sh 'npm run test-single-run -- --browsers PhantomJS'
    
    // archive karma test results (karma is configured to export junit xml files)
    step([$class: 'JUnitResultArchiver', 
            testResults: 'test-results/**/test-results.xml'])

//parallel phantomJS: {
  //  runTests("PhantomJS")
//}
    notify("Deploy to Staging")
}

input 'Deploy to Staging?'

stage name: 'Deploy', concurrency: 1
node {
    sh "echo '<h1>${env.BUILD_DISPLAY_NAME}</h1>' >> app/index.html"
    sh "rm -rf /var/jenkins_home/jobs/solitaire_pipeline/dockerFiles/*"
    sh "cp Dockerfile /var/jenkins_home/jobs/solitaire_pipeline/dockerFiles/Dockerfile"
    sh "cp -a app /var/jenkins_home/jobs/solitaire_pipeline/dockerFiles/"
    sh "ssh Gaz@192.168.1.3 'env PATH=/usr/local/bin docker build -t solitaire:latest ~/jenkins_home/jobs/solitaire_pipeline/dockerFiles'"
    sh "ssh Gaz@192.168.1.3 'env PATH=/usr/local/bin docker run -P -d solitaire'"
}

//chrome: {
  //  runTests("Chrome")
//}, firefox: {
 //   runtTests("Firefox")
//}, safari: {
  //  runTests("Safari")
//}

def runTests(browser){
    node {
        sh 'rm -rf *'
        unstash 'everything'
        sh "npm run test-single-run -- --browsers ${browser}"
        step([$class: 'JUnitResultArchiver', 
            testResults: 'test-results/**/test-results.xml'])
    }    
}

def notify(status){
    emailext (
      to: "wesmdemos@gmail.com",
      subject: "${status}: Job '${env.JOB_NAME} [${env.BUILD_NUMBER}]'",
      body: """<p>${status}: Job '${env.JOB_NAME} [${env.BUILD_NUMBER}]':</p>
        <p>Check console output at <a href='${env.BUILD_URL}'>${env.JOB_NAME} [${env.BUILD_NUMBER}]</a></p>""",
    )
}
