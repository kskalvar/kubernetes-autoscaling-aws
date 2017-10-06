Kubernetes Autoscaling on AWS
Kirk Kalvar
Updated 10/06/2017

# setup times
Provision 2 node cluster, including master: 10 minutes
Scaling Up: 20 minutes total time to scale to 10 pods and 4 nodes
Scaling Down: 10 minutes total time for 1 pod and 2 nodes

# enable heapster
kubectl create -f https://raw.githubusercontent.com/kubernetes/kops/master/addons/monitoring-standalone/v1.7.0.yaml

# check heapster enabled
kubectl get pods --namespace=kube-system

# create service & deployment
kubectl run php-apache --image=gcr.io/google_containers/hpa-example --requests=cpu=200m --expose --port=80

# show pods running
kubectl get pods --output wide

# show deployment
kubectl get deployment

# show service
kubectl get service

# autoscale deployment
kubectl autoscale deployment php-apache --cpu-percent=50 --min=2 --max=10

# show hpa
kubectl get hpa

# Modify AWS Autoscaling Group
# Be sure to enable AWS Metrics

# increase nodes autoscaling group Instances Max: 5

# Scaling Policies/Add policy

Name:  Kube Scale Up
Execute policy when: Kube Cluster >= 30 Percent	
breaches the alarm threshold: CPUUtilization >= 30 for 300 seconds
Take the action: Add 1 instances
And then wait: 60 seconds

Name: Kube Scale Down
Execute policy when:  Kube Cluster <= 10 Percent
breaches the alarm threshold: CPUUtilization <= 10 for 300 seconds
Take the action: Remove 1 instances 
And then wait: 60 seconds

# create load generator
kubectl run -i --tty load-generator --image=busybox /bin/sh

# test load
wget -q -O- http://php-apache.default.svc.cluster.local

# run load
while true; do wget -q -O- http://php-apache.default.svc.cluster.local; done

# stop load test
kubectl delete deployment load-generator

# stop autoscaling application
kubectl delete hpa,service,deployment php-apache
