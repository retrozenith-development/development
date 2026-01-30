pipeline {
    agent {
        node {
            label 'AOSP'
            customWorkspace '/aosp/crDroid'
        }
    }

    environment {
        // Set up ccache to speed up builds
        USE_CCACHE = '1'
        CCACHE_EXEC = '/usr/bin/ccache'
        // Adjust CCACHE_DIR to your Jenkins agent's ccache location
        CCACHE_DIR = '/var/lib/jenkins/.ccache'
    }

    options {
        // Timeout after 12 hours to prevent stuck builds
        timeout(time: 12, unit: 'HOURS')
        // Keep timestamps in logs
        timestamps()
    }

    stages {
        stage('Sync') {
            steps {
                script {
                    // Sync the repo
                    // Assumes the workspace is already initialized with `repo init`
                    sh 'repo sync -c -j$(nproc) --force-sync --no-clone-bundle --no-tags'
                }
            }
        }

        stage('Build') {
            steps {
                script {
                    sh '''
                        #!/bin/bash
                        # Initialize the build environment
                        source build/envsetup.sh
                        
                        # Set up ccache size (optional, can be done once manually)
                        ccache -M 50G

                        # Build for r8q using brunch (handles lunch + mka bacon)
                        brunch r8q
                    '''
                }
            }
        }
    }

    post {
        success {
            // Archive the artifacts as requested
            archiveArtifacts artifacts: 'out/target/product/r8q/crDroid*.zip, out/target/product/r8q/recovery.img, out/target/product/r8q/boot.img, out/target/product/r8q/super_empty.img, out/target/product/r8q/vbmeta.img', fingerprint: true
        }
        failure {
            echo 'Build failed!'
        }
    }
}
