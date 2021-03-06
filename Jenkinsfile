node() {
    deleteDir()
    // Create a workspace path.  We need this to be 79 chars max, otherwise some nodes fail.
    // The workspace path varies by node so get that path, and then add on 10 chars of a UUID string.
    ws_path = "$WORKSPACE".substring(0, "$WORKSPACE".indexOf("workspace/") + "workspace/".length()) + UUID.randomUUID().toString().substring(0, 10)
    ws(ws_path) {
        // pull the code
        dir('tools') {
            checkout scm
        }

        stage('Install dependencies') {
            sh """
            #!/bin/bash -x
            set -e
            # don't know OS, so trying both apt-get and yum install
            sudo apt-get install -y python3-dev || sudo yum install -y python36-devel.x86_64

            # virtualenv 16.3.0 is broken do not use it
            sudo python2 -m pip install --force-reinstall --upgrade pip virtualenv!=16.3.0 tox
            sudo python3 -m pip install --force-reinstall --upgrade pip virtualenv!=16.3.0 tox
            """
        }
        stage('Style tests') {
            sh """
            #!/bin/bash -x
            set -e

            cd tools/tensorflow_quantization
            make lint
            """
        }
        stage('Unit tests') {
            sh """
            #!/bin/bash -x
            set -e

            cd tools/tensorflow_quantization
            make unit_test
            """
        }

        stage('Integration tests') {
            sh """
            #!/bin/bash -x
            set -e

            cd tools/tensorflow_quantization
            make integration_test
            """
        }

        stage('C++ tests') {
            sh """
            #!/bin/bash -x
            set -e

            docker run --rm -e https_proxy -e http_proxy -e HTTPS_PROXY -e HTTP_PROXY -e no_proxy -e NO_PROXY quantization:latest /bin/bash -c "bazel test tensorflow/tools/graph_transforms:all"
            """
        }
    }
}
