# Install KubeSphere on Existing Kubernetes Cluster

> English | [中文](README_zh.md)

In addition to supporting deploying on VM and BM, KubeSphere also supports installing on cloud-hosted and on-premises existing Kubernetes clusters.

## 备注
该项目的核心文件包括
> - controller/installRunner.py 对应ks-installer docker镜像中（可以参考Dockerfile）的/hooks/kubesphere/installRunner.py（由shell-operator）监听并执行的存放脚本的目录
> - roles对应ks-installer docker镜像中（可以参考Dockerfile）的/kubesphere/installer/roles，由installRunner.py脚本中ansible_runner.run()函数执行的ansible的role
> - env对应ks-installer docker镜像中（可以参考Dockerfile）的/kubesphere/results/env，由installRunner.py脚本中ansible_runner.run()函数指定参数private_data_dir（代表ansible_runner默认加载配置文件的目录），其中env/cmdline与env/extravars默认加载，env/cmdline内容为"-b -e @/kubesphere/config/ks-config.yaml"，-e参数代表--extra-vars, @代表yaml文件中的[变量](https://docs.ansible.com/ansible/latest/user_guide/playbooks_variables.html)；env/extravars内容为"ansible_connection: local"。会在ansible-playbook -b -e @/kubesphere/config/ks-config.yaml -e @/kubesphere/results/env/extravars /kubesphere/playbooks/preinstall.yaml执行
> - playbooks对应ks-installer docker镜像中（可以参考Dockerfile）的/kubesphere/playbooks，由installRunner.py脚本中ansible_runner.run()函数指定参数playbook的目录，然后执行这个playbook

整个项目部署kubesphere的逻辑是，由shell-operator监听并执行installRunner.py脚本，然后通过installRunner.py脚本中的ansible_runner.run()函数来执行相应的playbook，然后执行playbook中对应的roles，roles中包含对应的组件（由configFile = '/kubesphere/config/ks-config.yaml（ks-config.yaml由installRunner.py中执行kubectl get cm，也就是kubesphere-minimal.yaml中的configMap对象生成）'获得）
至于ks-installer镜像中kubectl客户端可以执行k8s的操作，是由kubesphere-minimal.yaml文件中ClusterRole授权决定的
## Prerequisites

> - Kubernetes Version: 1.15.x, 1.16.x, 1.17.x;
> - Helm Version: `>= 2.10.0` (excluding 2.16.0 and 2.16.5) and < `3.0`, see [Install and Configure Helm in Kubernetes](https://devopscube.com/install-configure-helm-kubernetes/);
> - CPU > 1 Core, Memory > 2 G;
> - An existing default Storage Class in your Kubernetes clusters.
> - The CSR signing feature is activated in kube-apiserver when it is started with the `--cluster-signing-cert-file` and `--cluster-signing-key-file` parameters, see [RKE installation issue](https://github.com/kubesphere/kubesphere/issues/1925#issuecomment-591698309).

1. Make sure your Kubernetes version is compatible by running `kubectl version` in your cluster node. The output looks as the following:

```bash
$ kubectl version
Client Version: version.Info{Major:"1", Minor:"15", GitVersion:"v1.15.1", GitCommit:"4485c6f18cee9a5d3c3b4e523bd27972b1b53892", GitTreeState:"clean", BuildDate:"2019-07-18T09:09:21Z", GoVersion:"go1.12.5", Compiler:"gc", Platform:"linux/amd64"}
Server Version: version.Info{Major:"1", Minor:"15", GitVersion:"v1.15.1", GitCommit:"4485c6f18cee9a5d3c3b4e523bd27972b1b53892", GitTreeState:"clean", BuildDate:"2019-07-18T09:09:21Z", GoVersion:"go1.12.5", Compiler:"gc", Platform:"linux/amd64"}
```

> Note: Pay attention to `Server Version` line, if `GitVersion` is greater than `v1.15.0`, it's good to go. Otherwise you need to upgrade your kubernetes first.

2. Make sure you've already installed `Helm`, and it's version is greater than `2.10.0`. You can run `helm version` to check. The output looks like below:

```bash
$ helm version
Client: &version.Version{SemVer:"v2.13.1", GitCommit:"618447cbf203d147601b4b9bd7f8c37a5d39fbb4", GitTreeState:"clean"}
Server: &version.Version{SemVer:"v2.13.1", GitCommit:"618447cbf203d147601b4b9bd7f8c37a5d39fbb4", GitTreeState:"clean"}
```

> Note: If you get `helm: command not found`, it means `Helm` is not installed yet. You can refer to [Install Helm](https://helm.sh/docs/using_helm/#from-the-binary-releases) to find out how to install `Helm`, and don't forget to run `helm init` first after installation. If you use an older version (<2.10.0), you need to  [Upgrade Helm and Tiller](https://github.com/helm/helm/blob/master/docs/install.md#upgrading-tiller).

3. Check if the available resources meet the minimal prerequisite in your cluster.

```bash
$ free -g
              total        used        free      shared  buff/cache   available
Mem:              16          4          10           0           3           2
Swap:             0           0           0
```

4. Check if there is a default Storage Class in your cluster. An existing Storage Class is the prerequisite for KubeSphere installation.

```bash
$ kubectl get sc
NAME                      PROVISIONER               AGE
glusterfs (default)               kubernetes.io/glusterfs   3d4h
```

If your Kubernetes cluster environment meets all requirements mentioned above, then you can start to install KubeSphere.

## To Start Deploying KubeSphere

### Minimal Installation

> Attention: Following section is only used for minimal installation by default. KubeSphere has decoupled some core components since v2.1.0, for more pluggable components installation, see [Enable Pluggable Components](#enable-pluggable-components) and [Configuration Table](#configuration-table).

```bash
kubectl apply -f https://raw.githubusercontent.com/kubesphere/ks-installer/master/kubesphere-minimal.yaml
```

Then inspect the logs of installation.

```bash
kubectl logs -n kubesphere-system $(kubectl get pod -n kubesphere-system -l app=ks-install -o jsonpath='{.items[0].metadata.name}') -f
```

When all Pods of KubeSphere are running, it means the installation is successful. Check the port (30880 by default) of the console service by the following command. Then you can use `http://IP:30880` to access the console with the default account `admin/P@88w0rd`.

```bash
kubectl get svc/ks-console -n kubesphere-system
```

![KubeSphere Dashboard](https://pek3b.qingstor.com/kubesphere-docs/png/20191116004533.png)

### Enable Pluggable Components

> Attention: make sure there is enough CPU and memory available in your cluster. See the [Configuration Table](#configuration-table).

1. [Optional] Create the secret of certificate for Etcd in your Kubernetes cluster. This step is only needed when you want to enable Etcd monitoring.

> Note: Create the secret according to the actual Etcd certificate path of your cluster; If the Etcd has not been configured certificate, an empty secret needs to be created.

- If the Etcd has been configured with certificates, refer to the following step (The following command is an example that is only used for the cluster created by `kubeadm`):

```bash
$ kubectl -n kubesphere-monitoring-system create secret generic kube-etcd-client-certs  \
--from-file=etcd-client-ca.crt=/etc/kubernetes/pki/etcd/ca.crt  \
--from-file=etcd-client.crt=/etc/kubernetes/pki/etcd/healthcheck-client.crt  \
--from-file=etcd-client.key=/etc/kubernetes/pki/etcd/healthcheck-client.key
```

- If the Etcd has not been configured with certificates.

```bash
kubectl -n kubesphere-monitoring-system create secret generic kube-etcd-client-certs
```

2. If you already have a minimal KubeSphere setup, you still can enable the pluggable components by editing the ConfigMap of ks-installer using the following command.

> Note: Please make sure there is enough CPU and memory available in your cluster, see the [Configuration Table](#configuration-table) for reference.

```bash
kubectl edit cm ks-installer -n kubesphere-system
```

3. Inspect the logs of installation.

```bash
kubectl logs -n kubesphere-system $(kubectl get pod -n kubesphere-system -l app=ks-install -o jsonpath='{.items[0].metadata.name}') -f
```

## Upgrade

1. Download the Yaml file as follows:

```bash
wget https://raw.githubusercontent.com/kubesphere/ks-installer/master/kubesphere-minimal.yaml
```

2. Sync the changes from the old version to 2.1.1 in the config section of `kubesphere-minimal.yaml`, note the storage class and the pluggable components need to be consistent with the old version:

```bash
kubectl apply -f kubesphere-minimal.yaml
```

> Note: GitLab and Harbor are not included in 2.1.1, please refer to [Harbor Documentation](https://github.com/goharbor/harbor-helm) and [Gitlab Documentation](https://about.gitlab.com/install/) to install them if needed.

## Configuration Table

Pay attention to the resource request in the first column. You need to make sure there is enough CPU and memory available in your cluster, especially for enabling Logging, DevOps, Istio, Harbor and GitLab.

<table border=0 cellpadding=0 cellspacing=0 width=1288 style='border-collapse:
 collapse;table-layout:fixed;width:966pt'>
 <col width=202 style='mso-width-source:userset;mso-width-alt:7196;width:152pt'>
 <col width=232 style='mso-width-source:userset;mso-width-alt:8248;width:174pt'>
 <col width=595 style='mso-width-source:userset;mso-width-alt:21162;width:446pt'>
 <col class=xl6519753 width=259 style='mso-width-source:userset;mso-width-alt:
 9216;width:194pt'>
 <tr height=18 style='height:13.8pt'>
  <td colspan=2 height=18 class=xl6619753 width=434 style='height:13.8pt;
  width:326pt'>Parameter</td>
  <td class=xl6619753 width=595 style='width:446pt'>Description</td>
  <td class=xl6819753 width=259 style='width:194pt'>Default</td>
 </tr>
 <tr height=21 style='height:15.6pt'>
  <td height=21 class=xl6719753 style='height:15.6pt'>persistence</td>
  <td class=xl6719753>storageClass</td>
  <td class=xl1519753>Installer will use the default StorageClass, you can also designate another StorageClass</td>
  <td class=xl6519753>“”</td>
 </tr>
 <tr height=21 style='height:15.6pt'>
  <td rowspan=4 height=84 class=xl6719753 style='height:62.4pt'>etcd</td>
  <td class=xl6719753>monitoring</td>
  <td class=xl1519753>Whether to enable etcd monitoring</td>
  <td class=xl6519753>False</td>
 </tr>
 <tr height=21 style='height:15.6pt'>
  <td height=21 class=xl6719753 style='height:15.6pt'>endpointIps</td>
  <td class=xl1519753>etcd address（for etcd cluster, see an example value like `192.168.0.7,192.168.0.8,192.168.0.9`）</td>
  <td class=xl6519753></td>
 </tr>
 <tr height=21 style='height:15.6pt'>
  <td height=21 class=xl6719753 style='height:15.6pt'>port</td>
  <td class=xl1519753>etcd port (Default port: 2379, you can appoint any other port)</td>
  <td class=xl6519753>2379</td>
 </tr>
 <tr height=21 style='height:15.6pt'>
  <td height=21 class=xl6719753 style='height:15.6pt'>tlsEnable</td>
  <td class=xl1519753>Whether to enable etcd TLS certificate authentication.（True / False）</td>
  <td class=xl6519753>True</td>
 </tr>
 <tr height=21 style='height:15.6pt'>
  <td rowspan=5 height=105 class=xl6719753 style='height:78.0pt'>common</td>
  <td class=xl6719753>mysqlVolumeSize</td>
  <td class=xl1519753>MySQL volume size (cannot be modified after set)</td>
  <td class=xl6519753>20Gi</td>
 </tr>
 <tr height=21 style='height:15.6pt'>
  <td height=21 class=xl6719753 style='height:15.6pt'>minioVolumeSize</td>
  <td class=xl1519753>Minio volume size (cannot be modified after set)</td>
  <td class=xl6519753>20Gi</td>
 </tr>
 <tr height=21 style='height:15.6pt'>
  <td height=21 class=xl6719753 style='height:15.6pt'>etcdVolumeSize</td>
  <td class=xl1519753>etcd volume size (cannot be modified after set)</td>
  <td class=xl6519753>20Gi</td>
 </tr>
 <tr height=21 style='height:15.6pt'>
  <td height=21 class=xl6719753 style='height:15.6pt'>openldapVolumeSize</td>
  <td class=xl1519753>openldap volume size (cannot be modified after set)</td>
  <td class=xl6519753>2Gi</td>
 </tr>
 <tr height=21 style='height:15.6pt'>
  <td height=21 class=xl6719753 style='height:15.6pt'>redisVolumSize</td>
  <td class=xl1519753>redis volume size (cannot be modified after set)</td>
  <td class=xl6519753>2Gi</td>
 </tr>
 <tr height=21 style='height:15.6pt'>
  <td rowspan=2 height=42 class=xl6719753 style='height:31.2pt'>console</td>
  <td class=xl6719753>enableMultiLogin</td>
  <td class=xl1519753>Whether to enable multiple point login of one account（True / False）</td>
  <td class=xl6519753>False</td>
 </tr>
 <tr height=21 style='height:15.6pt'>
  <td height=21 class=xl6719753 style='height:15.6pt'>port</td>
  <td class=xl1519753>Console Port（NodePort）</td>
  <td class=xl6519753>30880</td>
 </tr>
 <tr height=21 style='height:15.6pt'>
  <td rowspan=4 height=84 class=xl6719753 style='height:62.4pt'>monitoring</td>
  <td class=xl6719753>prometheusReplicas</td>
  <td class=xl1519753>Prometheus replicas</td>
  <td class=xl6519753>1</td>
 </tr>
 <tr height=21 style='height:15.6pt'>
  <td height=21 class=xl6719753 style='height:15.6pt'>prometheusMemoryRequest</td>
  <td class=xl1519753>Prometheus memory request </td>
  <td class=xl6519753>400Mi</td>
 </tr>
 <tr height=21 style='height:15.6pt'>
  <td height=21 class=xl6719753 style='height:15.6pt'>prometheusVolumeSize</td>
  <td class=xl1519753>Prometheus volume size</td>
  <td class=xl6519753>20Gi</td>
 </tr>
 <tr height=21 style='height:15.6pt'>
  <td height=21 class=xl6719753 style='height:15.6pt'>grafana.enabled</td>
  <td class=xl1519753>Whether to enable Grafana installation（True / False）</td>
  <td class=xl6519753>False</td>
 </tr>
 <tr height=21 style='height:15.6pt'>
  <td height=21 class=xl6719753 style='height:15.6pt'>openpitrix </br>(at least 0.3 core, 300 MiB)</td>
  <td class=xl6719753>enable</td>
  <td class=xl1519753>App store and app templates are based on OpenPitrix, it's recommended to enable OpenPitrix installation（True / False）</td>
  <td class=xl6519753>False</td>
 <tr height=21 style='height:15.6pt'>
  <td rowspan=9 height=189 class=xl6619753 style='height:140.4pt'>logging</br>(at least 56 M, 2.76 G)</td>
  <td class=xl6719753>enabled</td>
  <td class=xl1519753>Whether to enable logging system installation<span
  style='mso-spacerun:yes'>&nbsp;&nbsp; </span>（True / False）</td>
  <td class=xl6519753>False</td>
 </tr>
 <tr height=21 style='height:15.6pt'>
  <td height=21 class=xl6719753 style='height:15.6pt'>elasticsearchMasterReplicas</td>
  <td class=xl1519753>Elasticsearch master replicas</td>
  <td class=xl6519753>1</td>
 </tr>
 <tr height=21 style='height:15.6pt'>
  <td height=21 class=xl6719753 style='height:15.6pt'>elasticsearchDataReplicas</td>
  <td class=xl1519753>Elasticsearch data replicas</td>
  <td class=xl6519753>1</td>
 </tr>
 <tr height=21 style='height:15.6pt'>
  <td height=21 class=xl6719753 style='height:15.6pt'>logsidecarReplicas</td>
  <td class=xl1519753>Logsidecar replicas</td>
  <td class=xl6519753>2</td>
 </tr>
 <tr height=21 style='height:15.6pt'>
  <td height=21 class=xl6719753 style='height:15.6pt'>elasticsearchVolumeSize</td>
  <td class=xl1519753>ElasticSearch volume size</td>
  <td class=xl6519753>20Gi</td>
 </tr>
 <tr height=21 style='height:15.6pt'>
  <td height=21 class=xl6719753 style='height:15.6pt'>logMaxAge</td>
  <td class=xl1519753>How many days the logs are remained </td>
  <td class=xl6519753>7</td>
 </tr>
 <tr height=21 style='height:15.6pt'>
  <td height=21 class=xl6719753 style='height:15.6pt'>elkPrefix</td>
  <td class=xl1519753>Log index<span style='mso-spacerun:yes'>&nbsp;</span></td>
  <td class=xl6519753>logstash<span style='mso-spacerun:yes'>&nbsp;</span></td>
 </tr>
 <tr height=21 style='height:15.6pt'>
  <td height=21 class=xl6719753 style='height:15.6pt'>containersLogMountedPath</td>
  <td class=xl1519753>The mounting path of container logs</td>
  <td class=xl6519753>“”</td>
 </tr>
 <tr height=21 style='height:15.6pt'>
  <td height=21 class=xl6719753 style='height:15.6pt'>kibana.enabled</td>
  <td class=xl1519753>Whether to enable Kibana installation<span style='mso-spacerun:yes'>&nbsp;
  </span>（True / False）</td>
  <td class=xl6519753>False</td>
 </tr>
 <tr height=21 style='height:15.6pt'>
  <td rowspan=8 height=168 class=xl6619753 style='height:124.8pt'>devops </br>(at least 0.47 core, 8.6  G for multi-node cluster)</td>
  <td class=xl6719753>enabled</td>
  <td class=xl1519753>Whether to enable DevOps system installation<span style='mso-spacerun:yes'>&nbsp;
  </span>（True / False）</td>
  <td class=xl6519753>False</td>
 </tr>
 <tr height=21 style='height:15.6pt'>
  <td height=21 class=xl6719753 style='height:15.6pt'>jenkinsMemoryLim</td>
  <td class=xl1519753>Jenkins Memory Limit</td>
  <td class=xl6519753>2Gi</td>
 </tr>
 <tr height=21 style='height:15.6pt'>
  <td height=21 class=xl6719753 style='height:15.6pt'>jenkinsMemoryReq</td>
  <td class=xl1519753>Jenkins Memory Request</td>
  <td class=xl6519753>1500Mi</td>
 </tr>
 <tr height=21 style='height:15.6pt'>
  <td height=21 class=xl6719753 style='height:15.6pt'>jenkinsVolumeSize</td>
  <td class=xl1519753>Jenkins volume size</td>
  <td class=xl6519753>8Gi</td>
 </tr>
 <tr height=21 style='height:15.6pt'>
  <td height=21 class=xl6719753 style='height:15.6pt'>jenkinsJavaOpts_Xms</td>
  <td class=xl1519753>Jenkins JVM parameter<span style='mso-spacerun:yes'>&nbsp;
  </span>（Xms）</td>
  <td class=xl6519753>512m</td>
 </tr>
 <tr height=21 style='height:15.6pt'>
  <td height=21 class=xl6719753 style='height:15.6pt'>jenkinsJavaOpts_Xmx</td>
  <td class=xl1519753>Jenkins<span style='mso-spacerun:yes'>&nbsp;
  </span>JVM parameter（Xmx）</td>
  <td class=xl6519753>512m</td>
 </tr>
 <tr height=21 style='height:15.6pt'>
  <td height=21 class=xl6719753 style='height:15.6pt'>jenkinsJavaOpts_MaxRAM</td>
  <td class=xl1519753>Jenkins<span style='mso-spacerun:yes'>&nbsp;
  </span>JVM parameter（MaxRAM）</td>
  <td class=xl6519753>2Gi</td>
 </tr>
 <tr height=21 style='height:15.6pt'>
  <td height=21 class=xl6719753 style='height:15.6pt'>sonarqube.enabled</td>
  <td class=xl1519753>Whether to install SonarQube（True / False）</td>
  <td class=xl6519753>False</td>
 </tr>
 <tr height=21 style='height:15.6pt'>
  <td height=21 class=xl6719753 style='height:15.6pt'>metrics_server </br>(at least 5 m, 44.35 MiB)</td>
  <td class=xl6719753>enabled</td>
  <td class=xl1519753>Whether to install metrics_server<span
  style='mso-spacerun:yes'>&nbsp;&nbsp;&nbsp; </span>（True / False）</td>
  <td class=xl6519753>False</td>
 </tr>
 <tr height=21 style='height:15.6pt'>
  <td height=21 class=xl6619753 style='height:15.6pt'>servicemesh</br>(at least 2 core, 3.6 G)</td>
  <td class=xl6719753>enabled</td>
  <td class=xl1519753>Whether to install Istio<span style='mso-spacerun:yes'>&nbsp;
  </span>（True / False）</td>
  <td class=xl6519753>False</td>
 </tr>
 <tr height=21 style='height:15.6pt'>
  <td height=21 class=xl6619753 style='height:15.6pt'>notification </br>(Notification and Alerting together, at least 0.08 core, 80 M)</td>
  <td class=xl6719753>enabled</td>
  <td class=xl1519753>Whether to install Notification sysytem （True / False）</td>
  <td class=xl6519753>False</td>
 </tr>
 <tr height=21 style='height:15.6pt'>
  <td height=21 class=xl6619753 style='height:15.6pt'>alerting</td>
  <td class=xl6719753>enabled</td>
  <td class=xl1519753>Whether to install Alerting sysytem （True / False）</td>
  <td class=xl6519753>False</td>
 </tr>
  <tr height=21 style='height:15.6pt'>
  <td height=21 class=xl6619753 style='height:15.6pt'>events</td>
  <td class=xl6719753>enabled</td>
  <td class=xl1519753>Whether to install events （True / False）</td>
  <td class=xl6519753>False</td>
 </tr>
 <![if supportMisalignedColumns]>
 <tr height=0 style='display:none'>
  <td width=202 style='width:152pt'></td>
  <td width=232 style='width:174pt'></td>
  <td width=595 style='width:446pt'></td>
  <td width=259 style='width:194pt'></td>
 </tr>
 <![endif]>
</table>

## Support, Discussion, and Community

If you need any help with KubeSphere, please join us at [Slack Channel](https://join.slack.com/t/kubesphere/shared_invite/enQtNTE3MDIxNzUxNzQ0LTZkNTdkYWNiYTVkMTM5ZThhODY1MjAyZmVlYWEwZmQ3ODQ1NmM1MGVkNWEzZTRhNzk0MzM5MmY4NDc3ZWVhMjE).
