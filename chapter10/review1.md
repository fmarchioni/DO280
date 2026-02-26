# Soluzione labs compreview-review

```bash
lab start compreview-review

mkdir -p ~/DO280/labs/compreview-review
oc login -u admin -p redhatocp https://api.ocp4.example.com:6443
oc adm groups new workshop-support
oc adm groups add-users workshop-support do280-support
oc adm groups new presenters
oc adm groups add-users presenters do280-presenter
oc adm groups new platform
oc adm groups add-users platform do280-platform
oc adm policy add-cluster-role-to-group admin workshop-support
cd DO280/labs/compreview-review/
vi groups-role.yaml
oc create -f groups-role.yaml
oc adm policy add-cluster-role-to-group manage-groups workshop-support
oc adm policy add-cluster-role-to-group cluster-admin platform
oc edit clusterrolebinding self-provisioners
oc login -u do280-attendee -p redhat
oc login -u admin -p redhatocp
oc new-project template-test
vi quota.yaml
oc create -f quota.yaml
vi limitrange.yaml
oc create -f limitrange.yaml
oc create deployment test-workload --image registry.ocp4.example.com:8443/redhattraining/hello-world-nginx:v1.0
oc get pods -o wide
oc debug --to-namespace="default" -- curl -s http://10.8.0.95:8080
oc label ns template-test workshop=template-test
vi networkpolicy.yaml
oc create -f networkpolicy.yaml

oc debug --to-namespace="default" -- curl -s http://10.8.0.95:8080
oc adm create-bootstrap-project-template -o yaml > project-template.yaml
oc get resourcequota/workshop limitrange/workshop networkpolicy/workshop -o yaml >> project-template.yaml
vi project-template.yaml
oc create -f project-template.yaml -n openshift-config
oc edit projects.config.openshift.io cluster
watch oc get pod -n openshift-apiserver
oc login -u do280-presenter -p redhat
oc new-project do280
oc get resourcequota/workshop limitrange/workshop networkpolicy/workshop
oc get project do280 -o yaml
oc describe rolebinding.rbac -n do280
oc login -u do280-support -p redhat
oc adm groups new do280-attendees
oc adm policy add-role-to-group edit do280-attendees -n do280
oc login -u do280-attendee -p redhat
oc login -u do280-support -p redhat
oc adm groups add-users do280-attendees do280-attendee
oc login -u do280-attendee -p redhat
oc create deployment attendee-workload --image registry.ocp4.example.com:8443/redhattraining/hello-world-nginx:v1.0
lab grade compreview-review
cd
lab grade compreview-review
```
