Steps involved to run the kubernetes in jenkins:
1. Install kubernetes plugin in jenkins
2. Create a namespace, service user, role, role binding and create token for the service account using the command `kubectl create token jenkins-svc-account -n jenkins --duration 100h > svc-account-token-expirable`. Store the same in credentials in the jenkins UI and use for the configuring the kubernetes cloud in jenkins UI.
3. Configure new kubernetes cloud configuration using above credential. For the 'Jenkins URL' field use the current jenkins master URL. If jenkins master is the local host machine, use the IP address for the same using the ipconfig command.
While configuring we get the certificate error when testing the connection. In that case it is due to self signed certificate. To resolve the same, extract the certificate of the kubernetes cluster (can be get from the .kube/config file and getting certificate encoded string for 'cluster' key)and then decode and store as .crt file. Once exported as .crt file use the command given to import the certificate as trusted certificate in java key store. `keytool -import -alias kubernetes-ca -keystore "C:\Program Files\Java\jdk-11\lib\security\cacerts" -file "C:\Users\yogtiwar2\local k8 cert\local-k8-ca-cert.crt" -storepass changeit`

4. Once the configuration is done, create a pod template for same same cloud configuration. You can speciify containers template in same pod template and then use that to create the jenkins agent pod. To stop container from stopping use the command `"/bin/sh" "-c"` with argument `"cat"`.

5. Use the `inheritFrom` property for kubernetes in the Jenkinsfile. Sample jenkins file is below.

```
pipeline {
    agent {
        kubernetes {
            inheritFrom 'jenkins-pod-template' // here the name is of the pod template you created in step 4.
        }
    }

    stages {
        stage('Hello') {
            steps {
                container('sfdx') //here container name is the name of the container you specified in the pod template created in step 4. {
                    echo 'Hello World'
                    sh "sf --version"
                }
            }
        }
    }
}
```
