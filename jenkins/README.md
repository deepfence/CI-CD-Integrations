# Jenkins example for Deepfence Vulnerability Scanner

This project demonstrates using Deepfence Vulnerability Scanner in Jenkins build pipeline.
After customer's image is built, Deepfence Vulnerability Scanner is run on the image and results are sent to Deepfence management console for further analysis.
There is also an option to fail the build in case number of vulnerabilities crosses given limit.

| Variable                           | Description                                               |
| ---------------------------------- | --------------------------------------------------------- |
| def deepfence_mgmt_console_ip = '' | Deepfence management console ip address                   |
| def fail_cve_count = 100           | Fail the build if number of vulnerabilities found >= this value. Set -1 to pass regardless of vulnerabilities.  | 

## Steps
- Ensure `deepfenceio/deepfence_agent` image is present in the vm where jenkins is installed.
```shell script
docker pull deepfenceio/deepfence_agent
```
- Add following jenkins stage to your Jenkinsfile
```
stage('Run Deepfence Vulnerability Scanner'){
    DeepfenceAgent = docker.image("deepfenceio/deepfence_agent:latest")
    try {
        c = DeepfenceAgent.run("-it --net=host -v /var/run/docker.sock:/var/run/docker.sock -v /var/lib/docker/:/fenced/mnt/host/var/lib/docker/:rw -v /:/fenced/mnt/host/:ro", "-a ${deepfence_mgmt_console_ip} -f audit -t all -i ${full_image_name} -e ${fail_cve_count}")
        sh "docker logs -f ${c.id}"
        def out = sh script: "docker inspect ${c.id} --format='{{.State.ExitCode}}'", returnStdout: true
        sh "exit ${out}"
    } finally {
        c.stop()
    }
}
```
- Set `deepfence_mgmt_console_ip`, `fail_cve_count` variables in Jenkinsfile