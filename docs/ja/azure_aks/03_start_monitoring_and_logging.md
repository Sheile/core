# 3. Azure AKSでモニターリング＆ロギングの開始

## 環境構築

1. RoboticBase/coreのclone先を、プロジェクトのルートディレクトリに設定

    ```
    $ export CORE_ROOT=${HOME}/core
    ```

1. 環境設定の読み込み

    ```
    $ source ${CORE_ROOT}/docs/azure_aks/env
    ```

## cygnus-elasticsearchを起動

1. cygnus-elasticsearch-serviceのインストール

    ```
    $ kubectl apply -f cygnus/cygnus-elasticsearch-service.yaml
    ```

    - 実行結果（例）

      ```
      service/cygnus-elasticsearch created
      ```

1. cygnus-elasticsearch-deploymentのインストール

    ```
    $ kubectl apply -f cygnus/cygnus-elasticsearch-deployment.yaml
    ```

    - 実行結果（例）

      ```
      deployment.apps/cygnus-elasticsearch created
      ```

1. cygnus-elasticsearchのインストール確認

    ```
    $ kubectl get pods -l app=cygnus-elasticsearch
    ```

    - 実行結果（例）

      ```
      NAME                                    READY   STATUS    RESTARTS   AGE
      cygnus-elasticsearch-8575567db7-g7rb4   1/1     Running   0          2m13s
      cygnus-elasticsearch-8575567db7-jjnvt   1/1     Running   0          2m13s
      cygnus-elasticsearch-8575567db7-v5lb8   1/1     Running   0          2m13s
      ```

1. cygnus-elasticsearchのサービス確認

    ```
    $ kubectl get services -l app=cygnus-elasticsearch
    ````

    - 実行結果（例）

      ```
      NAME                   TYPE        CLUSTER-IP     EXTERNAL-IP   PORT(S)             AGE
      cygnus-elasticsearch   ClusterIP   10.0.120.210   <none>        5050/TCP,8081/TCP   6m28s
      ```


## prometheusとgrafanaの起動

### coreos/prometheus-operatorのインストール

1. coreos/prometheus-operatorのレポジトリを登録

    ```
    $ helm repo add coreos https://s3-eu-west-1.amazonaws.com/coreos-charts/stable/
    ```

    - 実行結果（例）

      ```
      "coreos" has been added to your repositories
      ```

1. coreos/prometheus-operatorのインストール

    ```
    $ helm install coreos/prometheus-operator --name po --namespace monitoring
    ```

    - 実行結果（例）

      ```
      NAME:   po
      LAST DEPLOYED: Thu Feb 21 16:41:53 2019
      NAMESPACE: monitoring
      STATUS: DEPLOYED

      RESOURCES:
      ==> v1beta1/Deployment
      NAME                    DESIRED  CURRENT  UP-TO-DATE  AVAILABLE  AGE
      po-prometheus-operator  1        1        1           1          2m

      ==> v1beta1/PodSecurityPolicy
      NAME                    PRIV   CAPS  SELINUX   RUNASUSER  FSGROUP    SUPGROUP   READONLYROOTFS  VOLUMES
      po-prometheus-operator  false  []    RunAsAny  RunAsAny   MustRunAs  MustRunAs  false           [configMap emptyDir projected secret downwardAPI persistentVolumeClaim]

      ==> v1/ConfigMap
      NAME                    DATA  AGE
      po-prometheus-operator  1     2m

      ==> v1/ServiceAccount
      NAME                    SECRETS  AGE
      po-prometheus-operator  1        2m

      ==> v1beta1/ClusterRole
      NAME                        AGE
      psp-po-prometheus-operator  2m
      po-prometheus-operator      2m

      ==> v1beta1/ClusterRoleBinding
      NAME                        AGE
      psp-po-prometheus-operator  2m
      po-prometheus-operator      2m

      NOTES:
      The Prometheus Operator has been installed. Check its status by running:
        kubectl --namespace monitoring get pods -l "app=prometheus-operator,release=po"

      Visit https://github.com/coreos/prometheus-operator for instructions on how
      to create & configure Alertmanager and Prometheus instances using the Operator.
      ```

1. coreos/prometheus-operatorのインストール確認

    ```
    $ kubectl --namespace monitoring get pods -l "app=prometheus-operator,release=po"
    ```

    - 実行結果（例）

      ```
      NAME                                         READY   STATUS      RESTARTS   AGE
      po-prometheus-operator-56956994f-tnqth       1/1     Running     0          5m40s
      ```

1. coreos/kube-prometheusのインストール

    ```
    $ helm install coreos/kube-prometheus --name kp --namespace monitoring -f monitoring/kube-prometheus-azure.yaml
    ```

    - 実行結果（例）

      ```
      NAME:   kp
      LAST DEPLOYED: Thu Feb 21 17:07:49 2019
      NAMESPACE: monitoring
      STATUS: DEPLOYED

      RESOURCES:
      ==> v1beta1/DaemonSet
      NAME              DESIRED  CURRENT  READY  UP-TO-DATE  AVAILABLE  NODE-SELECTOR  AGE
      kp-exporter-node  4        4        0      4           0          <none>         4s

      ==> v1beta1/Deployment
      NAME                    DESIRED  CURRENT  UP-TO-DATE  AVAILABLE  AGE
      kp-grafana              1        1        1           0          4s
      kp-exporter-kube-state  1        1        1           0          4s

      ==> v1/Alertmanager
      NAME  KIND
      kp    Alertmanager.v1.monitoring.coreos.com

      ==> v1/ServiceAccount
      NAME                    SECRETS  AGE
      kp-exporter-node        1        5s
      kp-exporter-kube-state  1        5s
      kp-grafana              1        5s
      kp-prometheus           1        4s

      ==> v1beta1/ClusterRoleBinding
      NAME                        AGE
      psp-kp-grafana              4s
      psp-kp-exporter-kube-state  4s
      kp-prometheus               4s
      kp-exporter-kube-state      4s
      psp-kp-alertmanager         4s
      psp-kp-exporter-node        4s
      psp-kp-prometheus           4s

      ==> v1beta1/Role
      NAME                    AGE
      kp-exporter-kube-state  4s

      ==> v1/ServiceMonitor
      NAME                                 KIND
      kp-exporter-kube-scheduler           ServiceMonitor.v1.monitoring.coreos.com
      kp-exporter-kubelets                 ServiceMonitor.v1.monitoring.coreos.com
      kp-alertmanager                      ServiceMonitor.v1.monitoring.coreos.com
      kp-exporter-kube-etcd                ServiceMonitor.v1.monitoring.coreos.com
      kp-exporter-coredns                  ServiceMonitor.v1.monitoring.coreos.com
      kp-grafana                           ServiceMonitor.v1.monitoring.coreos.com
      kp-exporter-node                     ServiceMonitor.v1.monitoring.coreos.com
      kp-exporter-kubernetes               ServiceMonitor.v1.monitoring.coreos.com
      kp-exporter-kube-state               ServiceMonitor.v1.monitoring.coreos.com
      kp-prometheus                        ServiceMonitor.v1.monitoring.coreos.com
      kp-exporter-kube-controller-manager  ServiceMonitor.v1.monitoring.coreos.com

      ==> v1/Service
      NAME                                 CLUSTER-IP    EXTERNAL-IP  PORT(S)    AGE
      kp-exporter-kube-scheduler           None          <none>       10251/TCP  4s
      kp-exporter-coredns                  None          <none>       9153/TCP   4s
      kp-prometheus                        10.0.238.197  <none>       9090/TCP   4s
      kp-grafana                           10.0.137.243  <none>       80/TCP     4s
      kp-exporter-kube-etcd                None          <none>       4001/TCP   4s
      kp-exporter-kube-state               10.0.196.239  <none>       80/TCP     4s
      kp-exporter-node                     10.0.35.21    <none>       9100/TCP   4s
      kp-exporter-kube-controller-manager  None          <none>       10252/TCP  4s
      kp-alertmanager                      10.0.79.224   <none>       9093/TCP   4s

      ==> v1/Prometheus
      NAME           KIND
      kp-prometheus  Prometheus.v1.monitoring.coreos.com

      ==> v1/ConfigMap
      NAME        DATA  AGE
      kp-grafana  10    5s

      ==> v1beta1/RoleBinding
      NAME                    AGE
      kp-exporter-kube-state  4s

      ==> v1/PrometheusRule
      NAME                                 KIND
      kp-exporter-kube-controller-manager  PrometheusRule.v1.monitoring.coreos.com
      kp-exporter-kubernetes               PrometheusRule.v1.monitoring.coreos.com
      kp-exporter-kube-etcd                PrometheusRule.v1.monitoring.coreos.com
      kp-kube-prometheus                   PrometheusRule.v1.monitoring.coreos.com
      kp-exporter-node                     PrometheusRule.v1.monitoring.coreos.com
      kp-exporter-kube-scheduler           PrometheusRule.v1.monitoring.coreos.com
      kp-alertmanager                      PrometheusRule.v1.monitoring.coreos.com
      kp-exporter-kube-state               PrometheusRule.v1.monitoring.coreos.com
      kp-exporter-kubelets                 PrometheusRule.v1.monitoring.coreos.com
      kp-prometheus-rules                  PrometheusRule.v1.monitoring.coreos.com

      ==> v1beta1/PodSecurityPolicy
      NAME                    PRIV   CAPS  SELINUX   RUNASUSER  FSGROUP    SUPGROUP   READONLYROOTFS  VOLUMES
      kp-alertmanager         false  []    RunAsAny  RunAsAny   MustRunAs  MustRunAs  false           [configMap emptyDir projected secret downwardAPI persistentVolumeClaim]
      kp-prometheus           false  []    RunAsAny  RunAsAny   MustRunAs  MustRunAs  false           [configMap emptyDir projected secret downwardAPI persistentVolumeClaim]
      kp-grafana              false  []    RunAsAny  RunAsAny   MustRunAs  MustRunAs  false           [configMap emptyDir projected secret downwardAPI persistentVolumeClaim hostPath]
      kp-exporter-kube-state  false  []    RunAsAny  RunAsAny   MustRunAs  MustRunAs  false           [configMap emptyDir projected secret downwardAPI persistentVolumeClaim]
      kp-exporter-node        false  []    RunAsAny  RunAsAny   MustRunAs  MustRunAs  false           [configMap emptyDir projected secret downwardAPI persistentVolumeClaim hostPath]

      ==> v1/Secret
      NAME             TYPE    DATA  AGE
      kp-grafana       Opaque  2     5s
      alertmanager-kp  Opaque  1     5s

      ==> v1beta1/ClusterRole
      NAME                        AGE
      kp-exporter-kube-state      4s
      psp-kp-grafana              4s
      kp-prometheus               4s
      psp-kp-exporter-node        4s
      psp-kp-alertmanager         4s
      psp-kp-prometheus           4s
      psp-kp-exporter-kube-state  4s

      NOTES:
      DEPRECATION NOTICE:

      - alertmanager.ingress.fqdn is not used anymore, use alertmanager.ingress.hosts []
      - prometheus.ingress.fqdn is not used anymore, use prometheus.ingress.hosts []
      - grafana.ingress.fqdn is not used anymore, use prometheus.grafana.hosts []

      - additionalRulesConfigMapLabels is not used anymore, use additionalRulesLabels
      - prometheus.additionalRulesConfigMapLabels is not used anymore, use additionalRulesLabels
      - alertmanager.additionalRulesConfigMapLabels is not used anymore, use additionalRulesLabels
      - exporter-kube-controller-manager.additionalRulesConfigMapLabels is not used anymore, use additionalRulesLabels
      - exporter-kube-etcd.additionalRulesConfigMapLabels is not used anymore, use additionalRulesLabels
      - exporter-kube-scheduler.additionalRulesConfigMapLabels is not used anymore, use additionalRulesLabels
      - exporter-kubelets.additionalRulesConfigMapLabels is not used anymore, use additionalRulesLabels
      - exporter-kubernetes.additionalRulesConfigMapLabels is not used anymore, use additionalRulesLabels
      ```

1. monitoringネームスペース内のdaemonsetsを確認

    ```
    $ kubectl get daemonsets --namespace monitoring
    ```

    - 実行結果（例）

      ```
      NAME               DESIRED   CURRENT   READY   UP-TO-DATE   AVAILABLE   NODE SELECTOR   AGE
      kp-exporter-node   4         4         4       4            4           <none>          3m10s
      ```

1. monitoringネームスペース内のdeploymentsを確認

    ```
    $ kubectl get deployments --namespace monitoring
    ```

    - 実行結果（例）

      ```
      NAME                     DESIRED   CURRENT   UP-TO-DATE   AVAILABLE   AGE
      kp-exporter-kube-state   1         1         1            1           54s
      kp-grafana               1         1         1            1           54s
      po-prometheus-operator   1         1         1            1           3m
      ```

1. monitoringネームスペース内のstatefulsetsを確認

    ```
    $ kubectl get statefulsets --namespace monitoring
    ```

    - 実行結果（例）

      ```
      NAME                       DESIRED   CURRENT   AGE
      alertmanager-kp            1         1         8m27s
      prometheus-kp-prometheus   1         1         8m26s
      ```

1. monitoringネームスペース内のpodsを確認

    ```
    $ kubectl get pods --namespace monitoring
    ```

    - 実行結果（例）

      ```
      NAME                                         READY   STATUS      RESTARTS   AGE
      alertmanager-kp-0                            2/2     Running     0          11m
      kp-exporter-kube-state-665bdbdd97-75cvw      2/2     Running     0          10m
      kp-exporter-node-6zprs                       1/1     Running     0          11m
      kp-exporter-node-hg4n4                       1/1     Running     0          11m
      kp-exporter-node-tq5rw                       1/1     Running     0          11m
      kp-exporter-node-w58rh                       1/1     Running     0          11m
      kp-grafana-6df99fd77f-bp97q                  2/2     Running     0          11m
      po-prometheus-operator-56956994f-tnqth       1/1     Running     0          37m
      po-prometheus-operator-create-sm-job-svtq7   0/1     Completed   0          37m
      po-prometheus-operator-get-crd-wbggw         0/1     Completed   0          36m
      prometheus-kp-prometheus-0                   3/3     Running     1          11m
      ```

1. monitoringネームスペース内のservicesを確認

    ```
    $ kubectl get services --namespace monitoring
    ```

    - 実行結果（例）

      ```
      NAME                     TYPE        CLUSTER-IP     EXTERNAL-IP   PORT(S)             AGE
      alertmanager-operated    ClusterIP   None           <none>        9093/TCP,6783/TCP   14m
      kp-alertmanager          ClusterIP   10.0.79.224    <none>        9093/TCP            14m
      kp-exporter-kube-state   ClusterIP   10.0.196.239   <none>        80/TCP              14m
      kp-exporter-node         ClusterIP   10.0.35.21     <none>        9100/TCP            14m
      kp-grafana               ClusterIP   10.0.137.243   <none>        80/TCP              14m
      kp-prometheus            ClusterIP   10.0.238.197   <none>        9090/TCP            14m
      prometheus-operated      ClusterIP   None           <none>        9090/TCP            14m
      ```

1. monitoringネームスペース内のpersistentvolumeclaimsを確認

    ```
    $ kubectl get persistentvolumeclaims --namespace monitoring
    ```

    - 実行結果（例）

      ```
      NAME                                                     STATUS   VOLUME                                     CAPACITY   ACCESS MODES   STORAGECLASS      AGE
      alertmanager-kp-db-alertmanager-kp-0                     Bound    pvc-ca53b27f-35af-11e9-a6d0-3eb5d27c5279   30Gi       RWO            managed-premium   16m
      prometheus-kp-prometheus-db-prometheus-kp-prometheus-0   Bound    pvc-ca7da88f-35af-11e9-a6d0-3eb5d27c5279   30Gi       RWO            managed-premium   16m
      ```

## kube-prometheus-exporter-kubeletsにパッチを適用

1. 接続方式をHTTPSからHTTPに切り替える

    ※Azure AKS上に構築されたkubeletsのServiceMonitorはHTTPS接続ができないため
    https://github.com/coreos/prometheus-operator/issues/926

    ```
    $ kubectl get servicemonitor --namespace monitoring kp-exporter-kubelets -o yaml | sed 's/https/http/' | kubectl replace -f -
     ```

    - 実行結果（例）

      ```
      $ servicemonitor.monitoring.coreos.com/kp-exporter-kubelets replaced
      ```


## apiserverのServiceMonitorを削除

1. kp-exporter-kubernetesを削除

    ※Azure AKS上に構築されたapiserverのServiceMonitorは直接接続ができないため
    https://github.com/coreos/prometheus-operator/issues/1522

    ```
    $ kubectl delete servicemonitor --namespace monitoring kp-exporter-kubernetes
    ```

    - 実行結果（例）

      ```
      servicemonitor.monitoring.coreos.com "kp-exporter-kubernetes" deleted
      ```

## prometheusのルールを編集

1. prometheusrulesの「kp-kube-prometheus」を編集

    ```
    $ kubectl edit prometheusrules --namespace monitoring kp-kube-prometheus
    ```

    ※コマンドを実行するとエディタが起動します  
    -が付いている部分を削除し保存してください

    ```
      for: 10m
      labels:
        severity: warning
    -   - alert: DeadMansSwitch
    -     annotations:
    -       description: This is a DeadMansSwitch meant to ensure that the entire Alerting
    -         pipeline is functional.
    -       summary: Alerting DeadMansSwitch
    -     expr: vector(1)
    -     labels:
    -       severity: none
        - expr: process_open_fds / process_max_fds
          record: fd_utilization
        - alert: FdExhaustionClose
    ```

1. prometheusのルール「kp-exporter-kube-controller-manager」を編集

    ```
    $ kubectl edit prometheusrules --namespace monitoring kp-exporter-kube-controller-manager
    ```

    ※コマンドを実行するとエディタが起動します  
    　-が付いている部分を削除、+が付いている部分を追加し保存してください

    ```
      spec:
        groups:
        - name: kube-controller-manager.rules
    -     rules:
    -      - alert: K8SControllerManagerDown
    -        annotations:
    -          description: There is no running K8S controller manager. Deployments and replication
    -            controllers are not making progress.
    -          runbook: https://coreos.com/tectonic/docs/latest/troubleshooting/controller-recovery.html#recovering-a-controller-manager
    -          summary: Controller manager is down
    -        expr: absent(up{job="kube-controller-manager"} == 1)
    -        for: 5m
    -        labels:
    -          severity: critical
    +     rules: []
    ```

1. prometheusのルール「kp-exporter-kube-scheduler」を編集

    ```
    $ kubectl edit prometheusrules --namespace monitoring kp-exporter-kube-scheduler
    ```

    ※コマンドを実行するとエディタが起動します  
    　-が付いている部分を削除、+が付いている部分を追加し保存してください

    ```
          labels:
            quantile: "0.5"
          record: cluster:scheduler_binding_latency_seconds:quantile
    -  - alert: K8SSchedulerDown
    -    annotations:
    -      description: There is no running K8S scheduler. New pods are not being assigned
    -        to nodes.
    -      runbook: https://coreos.com/tectonic/docs/latest/troubleshooting/controller-recovery.html#recovering-a-scheduler
    -      summary: Scheduler is down
    -    expr: absent(up{job="kube-scheduler"} == 1)
    -    for: 5m
    -    labels:
    -      severity: critical
    ```

1. prometheusのルール「kp-exporter-kubernetes」を編集

    ```
    $ kubectl edit prometheusrules --namespace monitoring kp-exporter-kubernetes --namespace monitoring
    ```

    ※コマンドを実行するとエディタが起動します  
    　-が付いている部分を削除し保存してください

    ```
            for: 10m
            labels:
              severity: critical
      -    - alert: K8SApiserverDown
      -      annotations:
      -        description: No API servers are reachable or all have disappeared from service
      -          discovery
      -        summary: No API servers are reachable
      -      expr: absent(up{job="apiserver"} == 1)
      -      for: 20m
      -      labels:
      -        severity: critical
          - alert: K8sCertificateExpirationNotice
            annotations:
              description: Kubernetes API Certificate is expiring soon (less than 7 days)
    ```

## prometheusの確認

1. コマンドの作成

    ```
    $ echo 'kubectl --namespace monitoring port-forward $(kubectl get pod --namespace monitoring -l prometheus=kube-prometheus -l app=prometheus -o template --template "{{(index .items 0).metadata.name}}") 9090:9090'
    ```

1. prometheusのポートフォワーディングを開始

    ```
    $ kubectl --namespace monitoring port-forward $(kubectl get pod --namespace monitoring -l prometheus=kube-prometheus -l app=prometheus -o template --template "{{(index .items 0).metadata.name}}") 9090:9090
    ```

    - 実行結果（例）

      ```
      Forwarding from 127.0.0.1:9090 -> 9090
      Forwarding from [::1]:9090 -> 9090
      ```

1. 操作端末のデスクトップウィンドウ(GUI画面)を開き、prometheusのWEB管理画面を表示する

    ```
    $ xdg-open http://localhost:9090
    ```

1. prometheusのWEB管理画面が表示されたことを確認

    * GitHubに手順書をマージする際に「prometheus001.png」を貼りつけます

1. 「Alert」を選択

    * 「prometheus002.png」を貼りつけます

1. すべてのアラートが「0 active」と表示されることを確認

    * 「prometheus003.png」を貼りつけます

1. メニューから「Status」「Targets」をクリック

    * 「prometheus004.png」を貼りつけます

1. すべての「State」が「UP」になっていることを確認

    * 「prometheus005.png」を貼りつけます

1. ブラウザを終了

1. port-forwardingを閉じる


## grafanaの確認

1. コマンドの作成

    ```
    $ echo 'kubectl --namespace monitoring port-forward $(kubectl get pod --namespace monitoring -l app=kp-grafana -o template --template "{{(index .items 0).metadata.name}}") 3000:3000'
    ```

1. grafanaのポートフォワーディングを開始

    ```
    $ kubectl --namespace monitoring port-forward $(kubectl get pod --namespace monitoring -l app=kp-grafana -o template --template "{{(index .items 0).metadata.name}}") 3000:3000
    ```

    - 実行結果（例）

      ```
      Forwarding from 127.0.0.1:3000 -> 3000
      Forwarding from [::1]:3000 -> 3000
      ```

1. 操作端末のデスクトップウィンドウ(GUI画面)を開き、grafanaのWEB管理画面を表示する

    ```
    $ xdg-open http://localhost:3000
    ```

1. grafanaのWEB管理画面が表示されたことを確認

    * grafana001.pngを貼りつけます

1. 「email or username」欄に「admin」、「password」欄に「admin」を入力し「Log In」をクリック

    * grafana002.pngを貼りつけます

1. パスワードの変更画面が表示されるため、新規パスワードを入力し「Save」をクリック

    * grafana003.pngを貼りつけます

1. ホーム画面が表示されることを確認

    * grafana004.pngを貼りつけます

1. 左側下部の歯車マークをクリックし「Configuration」メニューから「Data Sources」をクリック

    * grafana005.pngを貼りつけます

1. 「prometheus」をクリック

    * grafana006.pngを貼りつけます

1. 「HTTP」メニューから「URL」のテキストボックスに「http://kp-prometheus:9090」を入力

    * grafana007.pngを貼りつけます

1. 最後部の「Save & Test」をクリック

    * grafana008.pngを貼りつけます

1. 「Data source is working」というメッセージが表示されたことを確認

    * grafana009.pngを貼りつけます

1. ブラウザを終了

1. port-forwardingを閉じる


## Elasticsearch、fluentd、Kibanaの起動

### Elasticsearchの起動

1. elasticsearch-azure-serviceのインストール

    ```
    $ kubectl apply -f logging/elasticsearch-azure-service.yaml
    ```

    - 実行結果（例）

      ```
      service/elasticsearch-logging created
      ```

1. elasticsearch-azure-deploymentのインストール

    ```
    $ kubectl apply -f logging/elasticsearch-azure-deployment.yaml
    ```

    - 実行結果（例）

      ```
      serviceaccount/elasticsearch-logging created
      clusterrole.rbac.authorization.k8s.io/elasticsearch-logging created
      clusterrolebinding.rbac.authorization.k8s.io/elasticsearch-logging created
      statefulset.apps/elasticsearch-logging created
      ```

1. elasticsearch-loggingのインストール確認

    ```
    $ kubectl get statefulsets --namespace monitoring -l k8s-app=elasticsearch-logging
    ```

    - 実行結果（例）

      ```
      NAME                    DESIRED   CURRENT   AGE
      elasticsearch-logging   2         2         2m24s
      ```

    ```
    $ kubectl get pods --namespace monitoring -l k8s-app=elasticsearch-logging
    ```

    - 実行結果（例）

      ```
      NAME                      READY   STATUS    RESTARTS   AGE
      elasticsearch-logging-0   1/1     Running   0          4m46s
      elasticsearch-logging-1   1/1     Running   0          2m26s
      ```

1. elasticsearch-loggingのサービス確認

    ```
    $ kubectl get services --namespace monitoring -l k8s-app=elasticsearch-logging
    ```

    - 実行結果（例）

      ```
      NAME                    TYPE        CLUSTER-IP     EXTERNAL-IP   PORT(S)    AGE
      elasticsearch-logging   ClusterIP   10.0.148.167   <none>        9200/TCP   7m45s
      ```

1. elastic-searchの使用しているボリュームを確認

    ```
    $ kubectl get persistentvolumeclaims -n monitoring -l k8s-app=elasticsearch-logging
    ```

    - 実行結果（例）

      ```
      NAME                                            STATUS   VOLUME                                     CAPACITY   ACCESS MODES   STORAGECLASS      AGE
      elasticsearch-logging-elasticsearch-logging-0   Bound    pvc-8e751ff8-38c0-11e9-a6d0-3eb5d27c5279   64Gi       RWO            managed-premium   7m55s
      elasticsearch-logging-elasticsearch-logging-1   Bound    pvc-e1b53352-38c0-11e9-a6d0-3eb5d27c5279   64Gi       RWO            managed-premium   5m35s
      ```

1. シャードの割り当て設定を変更

    ```
    $ kubectl exec -it elasticsearch-logging-0 --namespace monitoring -- curl -H "Content-Type: application/json" -X PUT http://elasticsearch-logging:9200/_cluster/settings -d '{"transient": {"cluster.routing.allocation.enable":"all"}}'
    ```

    - 実行結果（例）

      ```
      {"acknowledged":true,"persistent":{},"transient":{"cluster":{"routing":{"allocation":{"enable":"all"}}}}}
      ```

### fluentdの起動

1. fluentd-es-configmapのインストール

    ```
    $ kubectl apply -f logging/fluentd-es-configmap.yaml
    ```

    - 実行結果（例）

      ```
      configmap/fluentd-es-config-v0.2.0 created
      ```

1. fluentd-es-dsのインストール

    ```
    $ kubectl apply -f logging/fluentd-es-ds.yaml
    ```

    - 実行結果（例）

      ```
      serviceaccount/fluentd-es created
      clusterrole.rbac.authorization.k8s.io/fluentd-es created
      clusterrolebinding.rbac.authorization.k8s.io/fluentd-es created
      daemonset.apps/fluentd-es-v2.4.0 created
      ```

1. fluentd-esのdaemonsetsを確認

    ```
    $ kubectl get daemonsets --namespace monitoring -l k8s-app=fluentd-es
    ```

    - 実行結果（例）

      ```
      NAME                DESIRED   CURRENT   READY   UP-TO-DATE   AVAILABLE   NODE SELECTOR   AGE
      fluentd-es-v2.4.0   4         4         4       4            4           <none>          113s
      ```

1. fluentd-esのインストール確認

    ```
    $ kubectl get pods --namespace monitoring -l k8s-app=fluentd-es
    ```

    - 実行結果（例）

      ```
      NAME                      READY   STATUS    RESTARTS   AGE
      fluentd-es-v2.4.0-26fhm   1/1     Running   0          2m30s
      fluentd-es-v2.4.0-5ftbp   1/1     Running   0          2m30s
      fluentd-es-v2.4.0-cvrn4   1/1     Running   0          2m30s
      fluentd-es-v2.4.0-pv5nh   1/1     Running   0          2m30s
      ```

### Kibanaの起動準備

1. kibana-serviceのインストール

    ```
    $ kubectl apply -f logging/kibana-service.yaml
    ```

    - 実行結果（例）

      ```
      service/kibana-logging created
      ```

1. kibana-deploymentのインストール

    ```
    $ kubectl apply -f logging/kibana-deployment.yaml
    ```

    - 実行結果（例）

      ```
      deployment.apps/kibana-logging created
      ```

1. kibana-loggingのインストール確認

    ```
    $ kubectl get pods --namespace monitoring -l k8s-app=kibana-logging
    ```

    - 実行結果（例）

      ```
      NAME                              READY   STATUS    RESTARTS   AGE
      kibana-logging-76ff7dbb49-mbzqd   1/1     Running   0          102s
      ```

### curatorの起動

1. curator-configmapのインストール

    ```
    $ kubectl apply -f logging/curator-configmap.yaml
    ```

    - 実行結果（例）

      ```
      configmap/curator-config created
      ```

1. curator-cronjobのインストール

    ```
    $ kubectl apply -f logging/curator-cronjob.yaml
    ```

    - 実行結果（例）

      ```
      cronjob.batch/elasticsearch-curator created
      ```

1. elasticsearch-curatorのインストール確認

    ```
    $ kubectl get cronjobs --namespace monitoring -l k8s-app=elasticsearch-curator
    ```

    - 実行結果（例）

      ```
      NAME                    SCHEDULE     SUSPEND   ACTIVE   LAST SCHEDULE   AGE
      elasticsearch-curator   0 18 * * *   False     0        <none>          2m28s
      ```


## Kibanaの起動

1. コマンドの作成

    ```
    $ echo 'kubectl --namespace monitoring port-forward $(kubectl get pod -l k8s-app=kibana-logging --namespace monitoring -o template --template "{{(index .items 0).metadata.name}}") 5601:5601'
    ```

1. Kibanaのポートフォワーディングを開始

    ```
    $ kubectl --namespace monitoring port-forward $(kubectl get pod -l k8s-app=kibana-logging --namespace monitoring -o template --template "{{(index .items 0).metadata.name}}") 5601:5601
    ```

    - 実行結果（例）

      ```
      Forwarding from 127.0.0.1:5601 -> 5601
      Forwarding from [::1]:5601 -> 5601
      ```

1. 操作端末のデスクトップウィンドウ(GUI画面)を開き、KibanaのWEB管理画面を表示する

    ```
    $ xdg-open http://localhost:5601/
    ```

1. KibanaのWEB管理画面が表示されたことを確認

    * kibana001.pngを貼りつけます

1. 左側メニューから「Management」をクリック

    * kibana002.pngを貼りつけます

1. 「Kibana」メニューから「Index Patterns」をクリック

    * kibana003.pngを貼りつけます

1. 「Step 1 of 2:Define index pattern」メニューから「Index pattern」のテキストボックスに「logstash-*」を入力

    * kibana004.pngを貼りつけます

1. 「Success!」のメッセージが表示されたことを確認し「Next step」をクリック

    * kibana005.pngを貼りつけます

1. 「Step 2 of 2:Configure settings」メニューから「Time Filter field name」のテキストボックスに「@timestamp」を選択し「Create Index pattern」をクリック

    * kibana006.pngを貼りつけます

1. 「logstash-*」が作成されたことを確認

    * kibana007.pngを貼りつけます

1. ブラウザを終了

1. port-forwardingを閉じる


## grafanaにelasticsearch dashboardを追加

1. コマンドの作成

    ```
    $ echo 'kubectl --namespace monitoring port-forward $(kubectl get pod --namespace monitoring -l app=kp-grafana -o template --template "{{(index .items 0).metadata.name}}") 3000:3000'
    ```

1. grafanaのポートフォワーディングを開始

    ```
    $ kubectl --namespace monitoring port-forward $(kubectl get pod --namespace monitoring -l app=kp-grafana -o template --template "{{(index .items 0).metadata.name}}") 3000:3000
    ```

    - 実行結果（例）

      ```
      Forwarding from 127.0.0.1:3000 -> 3000
      Forwarding from [::1]:3000 -> 3000
      ```

1. 操作端末のデスクトップウィンドウ(GUI画面)を開き、KibanaのWEB管理画面を表示する

    ```
    $ xdg-open http://localhost:3000
    ```

1. grafanaのWEB管理画面が表示されたことを確認

    * 1_grafana-02_Homeを貼りつけます

1. 左側メニューバーから歯車マークを選択し「Configuration」メニューの「Data Sources」をクリック

    * 2_grafana-02_DataSources.pngを貼りつけます

1. 「Configuration」メニューから「Add data source」をクリック

    * 3_grafana-02_Add.pngを貼りつけます

1. 表示されているdata source typeから「ElasticSearch」をクリック

    * 4_grafana-02_Type.pngを貼りつけます

1. 設定値の入力画面が表示されるので、下記の値を入力

    ```
    Name: elasticsearch
    URL: http://elasticsearch-logging:9200
    Access: Server(Default)
    Index name: logstash-*
    Time field name: @timestamp
    Version: 6.0+
    ```

1. その後、最後部の「Save & Test」をクリック

    * 5_grafana-02_Setting_001.png、5_grafana-02_Setting_002.png、5_grafana-02_Setting_003.pngを貼りつけます

1. 「Index OK. Time field name OK.」と表示されることを確認

    * 6_grafana-02_Messeage.pngを貼りつけます

1. ホーム画面に戻り、プラス(＋)マークをクリックし「Create」メニューから「Import」をクリック

    * 7_grafana-02_Create.pngを貼りつけます

1. 「Upload .json File」をクリック

    * 8_grafana-02_Upload.pngを貼りつけます

1. monitoring/dashboard_elasticsearch.jsonを選択

    * 9_grafana-02_SelectFile.pngを貼りつけます

1. 「Import」をクリック

    * 10_grafana-02_Import.pngを貼りつけます

1. ElasticSeartchのダッシュボード画面が表示されることを確認

    * 11_grafana-02_DashBoard.pngを貼りつけます

1. ブラウザを終了

1. port-forwardingを閉じる
