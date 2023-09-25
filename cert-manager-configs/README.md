1. created a [role / user in AWS land](https://cert-manager.io/docs/configuration/acme/dns01/route53/) 

2. Do OCP Cert stuff. Probs won't bother with the API side of it though as not exposed to outside world :shrug:
a. create secret `echo $AWS_SECRET_ACCESS_KEY | base64` and inject into the `secret.yaml`
    `oc apply  -f secret.yaml`

b. create a clusterissuer req using the correct domains and hosted zone_id from AWS ( see `REPLACE_ME` values). *use the public HostedZone id*. You should see a PK arrive to the cert-manager ns 
    `oc apply -f 1_ClusterIssuer.yaml`

c. create cert requests for *.apps
    `oc apply -f 2_IngressCert.yaml` 
verify it exists before next step .... 
    `oc get secret ingress-letsencrypt-cert -n openshift-ingress`

d. edit the router's volume to replace the `router-certs-default` with `ingress-letsencrypt-cert`
    `oc edit deployment router-default -n openshift-ingress`
    `watch oc get pods -n openshift-ingress`