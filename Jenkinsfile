/**
properties([
  parameters([
    string(defaultValue: '1.0', description: 'Current version number', name: 'VERSION'),
    text(defaultValue: '', description: 'A list of changes', name: 'CHANGES'),
    booleanParam(defaultValue: false, description: 'If build should be marked as pre-release', name: 'PRERELEASE'),
    string(defaultValue: 'pfeerick-rock64', description: 'GitHub username or organization', name: 'GITHUB_USER'),
    string(defaultValue: 'linux-rootfs', description: 'GitHub repository', name: 'GITHUB_REPO'),
  ])
])
*/

node('docker && linux-build') {
  timestamps {
    wrap([$class: 'AnsiColorBuildWrapper', colorMapName: 'xterm']) {
      stage "Environment"
      checkout scm

      def environment = docker.build('build-environment:build-rock64-rootfs-image', 'environment')

      environment.inside("--privileged -u 0:0") {
        withEnv([
          "USE_CCACHE=true",
          "VERSION=$VERSION",
          "RELEASE=$BUILD_NUMBER"
        ]) {
            stage 'Images'
            sh '''#!/bin/bash
              set +xe
              make
            '''
        }
  
        withEnv([
          "VERSION=$VERSION",
          "CHANGES=$CHANGES",
          "PRERELEASE=$PRERELEASE",
          "GITHUB_USER=$GITHUB_USER",
          "GITHUB_REPO=$GITHUB_REPO"
        ]) {
          stage 'Release'
          sh '''#!/bin/bash
            set -xe
            shopt -s nullglob

            github-release release \
                --tag "${VERSION}" \
                --name "$VERSION: $BUILD_TAG" \
                --description "${CHANGES}\n\n${BUILD_URL}" \
                --draft

            for file in *.xz; do
              github-release upload \
                  --tag "${VERSION}" \
                  --name "$(basename "$file")" \
                  --file "$file" &
            done

            wait

            if [[ "$PRERELEASE" == "true" ]]; then
              github-release edit \
                --tag "${VERSION}" \
                --name "$VERSION: $BUILD_TAG" \
                --description "${CHANGES}\n\n${BUILD_URL}" \
                --pre-release
            else
              github-release edit \
                --tag "${VERSION}" \
                --name "$VERSION: $BUILD_TAG" \
                --description "${CHANGES}\n\n${BUILD_URL}"
            fi
          '''
        }
      }
    }
  }
}
