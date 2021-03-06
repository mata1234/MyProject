
https://docs.bitnami.com/tutorials/process-data-spark-kubernetes/

https://itnext.io/kafka-on-kubernetes-the-strimzi-way-part-1-bdff3e451788

https://github.com/strimzi/strimzi-kafka-operator/tree/main/examples/kafka

https://dzone.com/articles/migrate-data-across-kafka-cluster-using-mirrormake

https://medium.com/geekculture/tracing-kafka-messages-on-k8s-with-strimzi-and-jaeger-5f2e737c69ea
https://github.com/gkoenig/strimzi-jaeger-eval
https://fluxcd.io/

-------OCI

oci ce cluster create-kubeconfig --cluster-id ocid1.cluster.oc1.ap-mumbai-1.aaaaaaaailrkzdaq2qjst4vboorlp6zgpqow7q5ay55fm4r6ycoe7xpcvlca --file $HOME/.kube/config --region ap-mumbai-1 --token-version 2.0.0 

to suppress these warnings and to remove those permissions
WARNING: Kubernetes configuration file is group-readable. This is insecure. Location: /home/mataprasad/.kube/config
WARNING: Kubernetes configuration file is world-readable. This is insecure. Location: /home/mataprasad/.kube/config

ls -al ~/.kube/config
chmod o-r ~/.kube/config
chmod g-r ~/.kube/config

kubectl create namespace my-ws

---------SPARK BITNAMI installation with PVC
helm repo add stable https://charts.helm.sh/stable
helm install nfs stable/nfs-server-provisioner \
  --set persistence.enabled=true,persistence.size=5Gi
  
kubectl apply -f https://raw.githubusercontent.com/mata1234/k8s/master/spark-pvc.yml

docker run -v /tmp:/tmp -it bitnami/spark -- find /opt/bitnami/spark/examples/jars/ -name spark-examples* -exec cp {} /tmp/my.jar \;

echo "how much wood could a woodpecker chuck if a woodpecker could chuck wood" > /tmp/test.txt

kubectl cp /tmp/my.jar spark-data-pod:/data/my.jar
kubectl cp /tmp/test.txt spark-data-pod:/data/test.txt

kubectl exec -it spark-data-pod -- ls -al /data

kubectl delete pod spark-data-pod
------------
helm repo add bitnami https://charts.bitnami.com/bitnami

helm install spark bitnami/spark -f https://raw.githubusercontent.com/mata1234/k8s/master/spark-chart.yml

##
export SERVICE_IP=$(kubectl get --namespace default svc spark-master-svc -o jsonpath="{.status.loadBalancer.ingress[0]['ip', 'hostname'] }")
  echo http://$SERVICE_IP:80

  export EXAMPLE_JAR=$(kubectl exec -ti --namespace default spark-worker-0 -- find examples/jars/ -name 'spark-example*\.jar' | tr -d '\r')
  export SUBMIT_IP=$(kubectl get --namespace default svc spark-master-svc -o jsonpath="{.status.loadBalancer.ingress[0]['ip', 'hostname'] }")

  kubectl run --namespace default spark-client --rm --tty -i --restart='Never' \
    --image docker.io/bitnami/spark:3.1.1-debian-10-r68 \
    -- spark-submit --master spark://$SUBMIT_IP:7077 \
    --deploy-mode cluster \
    --class org.apache.spark.examples.SparkPi \
    $EXAMPLE_JAR 1000
##
---------------------- 
Run the commands below to obtain the master IP and submit your application.

kubectl get svc -l "app.kubernetes.io/instance=spark,app.kubernetes.io/name=spark"

kubectl run --namespace default spark-client --rm --tty -i --restart='Never' \
    --image docker.io/bitnami/spark:3.0.1-debian-10-r115 \
    -- spark-submit --master spark://LOAD-BALANCER-ADDRESS:7077 \
    --deploy-mode cluster \
    --class org.apache.spark.examples.JavaWordCount \
   /data/my.jar /data/test.txt
   
kubectl run --namespace default spark-client --rm --tty -i --restart='Never' \
    --image docker.io/bitnami/spark:3.0.1-debian-10-r115 \
    -- spark-submit --master spark://129.151.45.166:7077 \
    --deploy-mode cluster \
    --class org.apache.spark.examples.JavaWordCount \
   /data/my.jar /data/test.txt
   
kubectl get pods -o wide | grep WORKER-NODE-ADDRESS
kubectl get pods -o wide | grep 10.244.0.133

kubectl exec -it WORKER-POD-NAME -- bash
kubectl exec -it spark-worker-0 -- bash

cd /opt/bitnami/spark/work
cat SUBMISSION-ID/stdout

exit
----------------------
OR connect to the master pod and open up pyspark shell

kubectl get pods
kubectl exec spark-master-0 -it -- pyspark
run below code on pyspark
>>
words = 'there are infinite ways to learn kubernetes and helm charts'
sc = SparkContext()
seq = words.split()
data = sc.parallelize(seq)
counts = data.map(lambda word: (word, 1)).reduceByKey(lambda a, b: a + b).collect()
dict(counts)
sc.stop()
>>
exit()
----------------------
./bin/spark-submit   --class org.apache.spark.examples.SparkPi   --master spark://<master-IP>:<master-cluster-port>   --deploy-mode cluster  ./examples/jars/spark-examples_2.11-2.4.3.jar   1000
----------------------


--------------Kafka strimzi installation
kubectl create -f https://operatorhub.io/install/stable/strimzi-kafka-operator.yaml

OR

helm repo add strimzi https://strimzi.io/charts/
helm repo list
helm repo update
helm search repo strimzi
helm install strimzi-kafka-0.23 strimzi/strimzi-kafka-operator


to delete deployment - helm uninstall strimzi-kafka


kubectl get deployments

kubectl apply -f https://raw.githubusercontent.com/mata1234/k8s/master/kafka-pvc.yml

kubectl apply -f kafka-pvc.yml

kubectl get pods

kubectl get crd | grep strimzi

kubectl get crd kafkas.kafka.strimzi.io -o jsonpath="{.spec.versions[*].name}{'\n'}"

kubectl get strimzi -o name

to delete the cluster - kubectl delete kafka.kafka.strimzi.io/my-kafka-cluster

kubectl get kafka

kubectl get statefulset

kubectl get pod

kubectl get kt

kubectl get configmap

kubectl get configmap/my-kafka-cluster-kafka-config -o yaml

kubectl get pvc

kubectl get svc

kubectl get secret

view data
kubectl get configmap/my-kafka-cluster-kafka-config -o yaml

kubectl apply -f kafka-topic.yml
OR
kubectl run kafka-topics -ti --image=strimzi/kafka:latest-kafka-2.6.0 --rm=true --restart=Never -- bin/kafka-topics.sh  --zookeeper my-kafka-cluster-zookeeper-client:2181 --create --replication-factor 1 --partitions 2 --topic my-topic

kubectl exec my-kafka-cluster-kafka-0 -it -- /opt/kafka/bin/kafka-topics.sh --describe --topic my-topic --bootstrap-server 0.0.0.0:9092

Create a producer Pod
kubectl run kafka-producer -ti --image=strimzi/kafka:latest-kafka-2.6.0 --rm=true --restart=Never -- bin/kafka-console-producer.sh --broker-list my-kafka-cluster-kafka-bootstrap:9092 --topic my-topic

create a consumer Pod:
kubectl run kafka-consumer -ti --image=strimzi/kafka:latest-kafka-2.6.0 --rm=true --restart=Never -- bin/kafka-console-consumer.sh --bootstrap-server my-kafka-cluster-kafka-bootstrap:9092 --topic my-topic --from-beginning

run CTRL+C to kill producer and consumer pod

-------------------------------
-- RUN spark-pi example using spark-operator

http://www.openkb.info/2021/04/how-to-use-spark-operator-to-run-spark.html

kubectl get serviceAccounts
kubectl get po
helm list --all-namespaces

helm repo add incubator https://charts.helm.sh/incubator
helm repo update

kubectl apply -f spark-rbac.yaml
kubectl create clusterrolebinding spark-role --clusterrole=edit --serviceaccount=default:spark --namespace=default


helm repo add spark-operator https://googlecloudplatform.github.io/spark-on-k8s-operator
helm install so spark-operator/spark-operator --namespace spark-operator --create-namespace --set serviceAccounts.spark.name=spark --set sparkJobNamespace=default
helm uninstall so --namespace spark-operator
kubectl get pods -n spark-operator
kubectl get deployment -n spark-operator
kubectl get clusterrolebinding |grep spark-operator
kubectl describe clusterrolebinding so-spark-operator
kubectl get role -n spark-operator
helm status --namespace spark-operator so

kubectl apply -f https://raw.githubusercontent.com/mata1234/k8s/master/spark-pi.yaml
kubectl apply -f spark-pi.yaml

kubectl get pods
kubectl get sparkapplications
kubectl get sparkapplications spark-pi -o=yaml
kubectl get sparkapplications spark-pi -o=yaml | grep service
kubectl describe sparkapplication spark-pi
kubectl delete sparkapplication spark-pi


kubectl port-forward spark-pi-1547981232122-driver 4040:4040

kubectl logs -f spark-pi-driver
kubectl logs --tail=n spark-pi-driver
kubectl describe pod spark-pi-driver
kubectl logs --tail=20 spark-pi-driver | grep 'Pi is'

cat <<'EOF'>> log4j.properties
log4j.rootCategory=DEBUG, console
log4j.appender.console=org.apache.log4j.ConsoleAppender
log4j.appender.console.target=System.err
log4j.appender.console.layout=org.apache.log4j.PatternLayout
log4j.appender.console.layout.ConversionPattern=%d{yy/MM/dd HH:mm:ss} %p %c{1}: %m%n    
EOF

kubectl create configmap spark-conf-map --from-file log4j.properties
kubectl create configmap spark-conf-map --from-file log4j.properties



------------------------------