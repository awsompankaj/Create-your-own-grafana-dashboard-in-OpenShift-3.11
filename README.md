# Create-your-own-grafana-dashboard-in-OpenShift-3.11

This is brief summary of steps which i performed to create an additional grafana dashboard in openshift 3.11.
OpenShift Container Platform ships with a pre-configured and self-updating monitoring stack that is based on the Prometheus open source project and its wider eco-system. It provides monitoring of cluster components and ships with a set of alerts to immediately notify the cluster administrator about any occurring problems and a set of Grafana dashboards. the complete monitoring stack resides in namespace openshift-monitoring.
This issue in grafana is that you can not edit/install or create any additional dashboard in openshift version 3.11. The monitoring stack ensures that the services running within namespace openshift-monitoring remain unchanged and there is no options available to import grafana dashboard from GUI or from CLI. for reference check issue with grafana dashboard in OCP 3.11.
so if you want to create your own custom grafana follow below steps.
Steps to be followed to create your own dashboard
I have created a new project named my-monitoring.
# oc new-project my-monitoring
create new projectExport secret grafana-datasources from openshift-monitoring namespace.
# oc get secrets grafana-datasources -n openshift-monitoring --export -o yaml > grafana-datasources.yaml
export secretscreate new secret using exported secret from openshift-monitoring namespace in your namespace my-monitoring
# oc create -f grafana-datasources.yaml
Now the important part check the grafana version which is running on your current monitoring stack.
grafana dashboardin our case it is grafana version 5.2.1
Now create a new-app using grafana docker image of same version 5.2.1
# oc new-app --name=mydashboard docker.io/grafana/grafana:5.2.1
Now add volume using secret grafana-datasources which we exported earlier.
# oc set volume dc/mydashboard --add --name=grafana-dashsources --type=secret --secret-name=grafana-datasourc
es --mount-path=/etc/grafana/provisioning/datasources --namespace=my-monitoring
create one more EmptyDir volume at /var/lib/grafana
# oc set volume dc/mydashboard --add --name=grafana-storage --mount-path=/var/lib/grafana --namespace=my-monitoring
verify both mount point are showing while you describe the deploymentconfig which is mydashboard in our case.
now expose the service
# oc expose svc/mydashboard
and join the namespace networks to connect to each other
#o c adm pod-network join-projects --to=my-monitoring openshift-monitoring
Now use your newly created grafana dashboard using the routes and put default credentials admin:admin.
Now import or create your own grafana dashboards.
Congratulations!! here is your own grafana dashboard
# Note: imported or newly created dashboards are not persistent and once the pod restarted you need to again import the dashboards, either you take the backup of dashboards or use dashboard json format files as configmaps in deploymentconfigs.

you can refer :https://medium.com/@Awsompankaj/create-your-own-openshift-dashboard-in-grafana-ocp-3-11-f51545f3a22e?source=friends_link&sk=38b1b220ee7d4f9970a7f447d60b83bd
