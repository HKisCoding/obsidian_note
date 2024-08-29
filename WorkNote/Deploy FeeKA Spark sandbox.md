
# smartpay_fms_drools
## 1. Get code from remote to 10.1.1.156
Jenkins: https://jenkins.paysmart.com.vn/login?from=%2F
User: dev
password: 

Data Analytics -> smartpay-fms -> dev 

- Choose Build with parameters
- Choose Branch: origin/sandbox

## 2. Sync code to 10.205.51.7/8/9
RU
``` shell
sudo su
su - hadoop

/smartpay/scripts/deploy_app.sh smartpay-fms
```

## 3. Run application:
Go to 1 node
``` shell
/smartpay/scripts/deploy_app.sh smartpay-fms SparkDroolsMDRKA
/smartpay/scripts/deploy_app.sh smartpay-fms SparkDroolsAccumulate
/smartpay/scripts/deploy_app.sh smartpay-fms SparkFmsQueueConsumer
/smartpay/scripts/deploy_app.sh smartpay-fms SparkDroolsFmsScheduler

```


# smartpay_fms
## 1. Get code from remote to 10.1.1.156
Jenkins: http://10.1.1.156:8090/job/sandbox/
Pipeline: deploy-lakehouse-fms-fee
Branchname: sandbox



## 2. Run pod
ssh: 10.1.1.156

``` bash
sudo su
su jenkins
docker exec -it jenkins /bin/bash
cd ~/jenkins-data/cicd/fms-fee/git/fms_fee/deployment/fms-fee/{{ service }}/overlays

kubectl apply -k . -n sandbox
```


# Update Drools drl 

## 1. Run build Jenkins
Jenkins: http://10.1.1.156:8090/job/sandbox/
Pipeline: deploy-lakehouse-fms-fee
Branchname: sandbox
service: buildFMSDrlSchemeUpdate

## 2. Change cronjob
login Jenkins:
``` shell
sudo su
su jenkins
vim ~/jenkins/jenkins-data/cicd/fms-fee/git/fms_fee/deployment/fms-fee/fms-drools-scheme-update/base/deployment.yaml
```

`"* 22 * * *"` -> `"*/1 * * * *"`

## 3. Run pod 


``` bash
docker exec -it jenkins /bin/bash
cd ~/cicd/fms-fee/git/fms_fee/deployment/fms-fee/fms-drools-scheme-update/overlays

kubectl apply -k . -n sandbox
```

## 4. Reset Cronjob
``` bash
git checkout -- jenkins/jenkins-data/cicd/fms-fee/git/fms_fee/deployment/fms-fee/fms-drools-scheme-update/base/deployment.yaml
```


