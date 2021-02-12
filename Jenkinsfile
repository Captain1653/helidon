/*
 * Copyright (c) 2020, 2021 Oracle and/or its affiliates.
 *
 * Licensed under the Apache License, Version 2.0 (the "License");
 * you may not use this file except in compliance with the License.
 * You may obtain a copy of the License at
 *
 *     http://www.apache.org/licenses/LICENSE-2.0
 *
 * Unless required by applicable law or agreed to in writing, software
 * distributed under the License is distributed on an "AS IS" BASIS,
 * WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
 * See the License for the specific language governing permissions and
 * limitations under the License.
 */

pipeline {
  agent {
    label "linux"
  }
  options {
    parallelsAlwaysFailFast()
  }
  environment {
    NPM_CONFIG_REGISTRY = credentials('npm-registry')
  }
  stages {
    stage('default-pipeline') {
      steps {
        script {
          runParallel([
            run('build',
              saveCache { sh './etc/scripts/build.sh' },
              {
                runParallel([
                  run('unit-tests',         withCache { test { sh './etc/scripts/test-unit.sh' }}),
                  run('integration-tests',  withCache { test { sh './etc/scripts/test-integ.sh' }}),
                  run('native-image-tests', withCache { test { sh './etc/scripts/test-integ-native-image.sh' }}),
                  run('tcks',               withCache { test { sh './etc/scripts/tcks.sh' }}),
                  run('javadocs',           withCache { sh './etc/scripts/javadocs.sh' }),
                  run('spotbugs',           withCache { sh './etc/scripts/spotbugs.sh' }),
                  run('javadocs',           withCache { sh './etc/scripts/javadocs.sh' }),
                  run('site',               withCache { sh './etc/scripts/site.sh' }),
                  run('archetypes',         withCache { sh './etc/scripts/archetypes.sh' })
                ])
              })
//             run('copyright', { sh './etc/scripts/copyright.sh' }),
//             run('checkstyle', { sh './etc/scripts/checkstyle.sh' })
          ])
        }
      }
    }
    stage('release-pipeline') {
      when { branch '**/release-*' }
      environment {
        GITHUB_SSH_KEY = credentials('helidonrobot-github-ssh-private-key')
        MAVEN_SETTINGS_FILE = credentials('helidonrobot-maven-settings-ossrh')
        GPG_PUBLIC_KEY = credentials('helidon-gpg-public-key')
        GPG_PRIVATE_KEY = credentials('helidon-gpg-private-key')
        GPG_PASSPHRASE = credentials('helidon-gpg-passphrase')
      }
      steps { sh './etc/scripts/release.sh release_build' }
    }
  }
}

def test(closure) {
  return {
    try { closure() } finally {
      archiveArtifacts artifacts: "**/target/surefire-reports/*.txt, **/target/failsafe-reports/*.txt"
      junit testResults: '**/target/surefire-reports/*.xml,**/target/failsafe-reports/*.xml'
    }
  }
}
def saveCache(closure) {
  return {
    closure()
    stash name: 'build-cache', includes: 'target/build-cache.tar'
  }
}
def withCache(closure) {
  return {
    unstash 'build-cache'
    closure()
  }
}
def run(name, ...closures) {
  return [ name: { node { stage(name) { closures.each { it() }}}} ]
}
def runParallel(stages) {
  return parallel(stages.collectEntries { it })
}