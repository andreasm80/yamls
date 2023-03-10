# Section for AMKO
Here I have collected the various YAML definitions I have been using to test out GSLB with Avi and AMKO.

## AMKO values.yaml (used in my blog post blog.andreasm.io)
```
# Default values for amko.
# This is a YAML-formatted file.
# Declare variables to be passed into your templates.

replicaCount: 1

image:
  repository: projects.registry.vmware.com/ako/amko
  pullPolicy: IfNotPresent

# Configs related to AMKO Federator
federation:
  # image repository
  image:
    repository: projects.registry.vmware.com/ako/amko-federator
    pullPolicy: IfNotPresent
  # cluster context where AMKO is going to be deployed
  currentCluster: 'k8slab-admin@k8slab'
  # Set to true if AMKO on this cluster is the leader
  currentClusterIsLeader: true
  # member clusters to federate the GSLBConfig and GDP objects on, if the
  # current cluster context is part of this list, the federator will ignore it
  memberClusters:
  - 'k8slab-admin@k8slab'
  - 'tkgs-cluster-1-admin@tkgs-cluster-1'

# Configs related to AMKO Service discovery
serviceDiscovery:
  # image repository
  # image:
  #   repository: projects.registry.vmware.com/ako/amko-service-discovery
  #   pullPolicy: IfNotPresent

# Configs related to Multi-cluster ingress. Note: MultiClusterIngress is a tech preview.
multiClusterIngress:
  enable: false

configs:
  gslbLeaderController: '172.18.5.51'
  controllerVersion: 22.1.1
  memberClusters:
  - clusterContext: 'k8slab-admin@k8slab'
  - clusterContext: 'tkgs-cluster-1-admin@tkgs-cluster-1'
  refreshInterval: 1800
  logLevel: INFO
  # Set the below flag to true if a different GSLB Service fqdn is desired than the ingress/route's
  # local fqdns. Note that, this field will use AKO's HostRule objects' to find out the local to global
  # fqdn mapping. To configure a mapping between the local to global fqdn, configure the hostrule
  # object as:
  # [...]
  # spec:
  #  virtualhost:
  #    fqdn: foo.avi.com
  #    gslb:
  #      fqdn: gs-foo.avi.com
  useCustomGlobalFqdn: true

gslbLeaderCredentials:
  username: ''
  password: ''

globalDeploymentPolicy:
  # appSelector takes the form of:
  appSelector:
    label:
      app: 'gslb' 
  # Uncomment below and add the required ingress/route/service label
  # appSelector:

  # namespaceSelector takes the form of:
  # namespaceSelector:
  #   label:
  #     ns: gslb   <example label key-value for namespace>
  # Uncomment below and add the reuqired namespace label
  # namespaceSelector:

  # list of all clusters that the GDP object will be applied to, can take any/all values
  # from .configs.memberClusters
  matchClusters:
  - cluster: 'k8slab-admin@k8slab'
  - cluster: 'tkgs-cluster-1-admin@tkgs-cluster-1'

  # list of all clusters and their traffic weights, if unspecified, default weights will be
  # given (optional). Uncomment below to add the required trafficSplit.
  # trafficSplit:
  #   - cluster: "cluster1-admin"
  #     weight: 8
  #   - cluster: "cluster2-admin"
  #     weight: 2

  # Uncomment below to specify a ttl value in seconds. By default, the value is inherited from
  # Avi's DNS VS.
  # ttl: 10

  # Uncomment below to specify custom health monitor refs. By default, HTTP/HTTPS path based health
  # monitors are applied on the GSs.
  # healthMonitorRefs:
  # - hmref1
  # - hmref2

  # Uncomment below to specify a Site Persistence profile ref. By default, Site Persistence is disabled.
  # Also, note that, Site Persistence is only applicable on secure ingresses/routes and ignored
  # for all other cases. Follow https://avinetworks.com/docs/20.1/gslb-site-cookie-persistence/ to create
  # a Site persistence profile.
  # sitePersistenceRef: gap-1

  # Uncomment below to specify gslb service pool algorithm settings for all gslb services. Applicable
  # values for lbAlgorithm:
  # 1. GSLB_ALGORITHM_CONSISTENT_HASH (needs a hashMask field to be set too)
  # 2. GSLB_ALGORITHM_GEO (needs geoFallback settings to be used for this field)
  # 3. GSLB_ALGORITHM_ROUND_ROBIN (default)
  # 4. GSLB_ALGORITHM_TOPOLOGY
  #
  # poolAlgorithmSettings:
  #   lbAlgorithm:
  #   hashMask:           # required only for lbAlgorithm == GSLB_ALGORITHM_CONSISTENT_HASH
  #   geoFallback:        # fallback settings required only for lbAlgorithm == GSLB_ALGORITHM_GEO
  #     lbAlgorithm:      # can only have either GSLB_ALGORITHM_ROUND_ROBIN or GSLB_ALGORITHM_CONSISTENT_HASH
  #     hashMask:         # required only for fallback lbAlgorithm as GSLB_ALGORITHM_CONSISTENT_HASH

serviceAccount:
  # Specifies whether a service account should be created
  create: true
  # Annotations to add to the service account
  annotations: {}
  # The name of the service account to use.
  # If not set and create is true, a name is generated using the fullname template
  name:

resources:
  limits:
    cpu: 250m
    memory: 300Mi
  requests:
    cpu: 100m
    memory: 200Mi

service:
  type: ClusterIP
  port: 80

rbac:
  # creates the pod security policy if set to true
  pspEnable: false

persistentVolumeClaim: ''
mountPath: /log
logFile: amko.log

federatorLogFile: amko-federator.log
```
## GSLB Kubernetes Config Secret
```
apiVersion: v1
clusters:
- cluster:
    certificate-authority-data: Base64
    server: https://10.170.0.10:6443
  name: k8slab
- cluster:
    certificate-authority-data:
    server: https://192.168.85.1:6443
  name: tkgs-cluster-1
contexts:
- context:
    cluster: k8slab
    user: k8slab-admin
  name: k8slab-admin@k8slab
- context:
    cluster: tkgs-cluster-1
    user: tkgs-cluster-1-admin
  name: tkgs-cluster-1-admin@tkgs-cluster-1
current-context: tkgs-cluster-1-admin@tkgs-cluster-1
kind: Config
preferences: {}
users:
- name: k8slab-admin
  user:
    client-certificate-data: Base64
    client-key-data: Base64
- name: tkgs-cluster-1-admin
  user:
    client-certificate-data: Base64
    client-key-data: Base64
```
## Apple test App site a
```
kind: Pod
apiVersion: v1
metadata:
  name: apple-app
  labels:
    app: apple
  namespace: fruit
spec:
  containers:
    - name: apple-app
      image: hashicorp/http-echo
      args:
        - "-text=This is Site A Home-LAB"

---

kind: Service
apiVersion: v1
metadata:
  name: apple-service
  namespace: fruit
spec:
  selector:
    app: apple
  ports:
    - port: 5678 # Default port for image
```
## Apple test App site b
```
kind: Pod
apiVersion: v1
metadata:
  name: apple-app
  labels:
    app: apple
  namespace: fruit
spec:
  containers:
    - name: apple-app
      image: hashicorp/http-echo
      args:
        - "-text=This is Site B - Remote Site"

---

kind: Service
apiVersion: v1
metadata:
  name: apple-service
  namespace: fruit
spec:
  selector:
    app: apple
  ports:
    - port: 5678 # Default port for image
```
## Banana test App site a
```
kind: Pod
apiVersion: v1
metadata:
  name: banana-app
  labels:
    app: banana
  namespace: fruit
spec:
  containers:
    - name: banana-app
      image: hashicorp/http-echo
      args:
        - "-text=This is Site A - Home-LAB"

---

kind: Service
apiVersion: v1
metadata:
  name: banana-service
  namespace: fruit
spec:
  selector:
    app: banana
  ports:
    - port: 5678 # Default port for image
```
## Banana test App site b
```
kind: Pod
apiVersion: v1
metadata:
  name: banana-app
  labels:
    app: banana
  namespace: fruit
spec:
  containers:
    - name: banana-app
      image: hashicorp/http-echo
      args:
        - "-text=This is Site B - Remote Site"

---

kind: Service
apiVersion: v1
metadata:
  name: banana-service
  namespace: fruit
spec:
  selector:
    app: banana
  ports:
    - port: 5678 # Default port for image
```
## GSLB Ingress Site A
```
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: ingress-example
  labels:
    app: gslb
  namespace: fruit
#  annotations:
#    ako.vmware.com/enable-tls: "true"

spec:
  ingressClassName: avi-lb
  rules:
    - host: fruit-global.guzware.net
      http:
        paths:
        - path: /apple
          pathType: Prefix
          backend:
            service:
              name: apple-service
              port:
                number: 5678
        - path: /banana
          pathType: Prefix
          backend:
            service:
              name: banana-service
              port:
                number: 5678
---
apiVersion: ako.vmware.com/v1alpha1
kind: HostRule
metadata:
  namespace: fruit
  name: gslb-host-rule-fruit
spec:
  virtualhost:
    fqdn: fruit-global.guzware.net
    enableVirtualHost: true
    gslb:
      fqdn: fruit.gslb.guzware.net
```
## GSLB Ingress Site B
```
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: ingress-example
  labels:
    app: gslb
  namespace: fruit
#  annotations:
#    ako.vmware.com/enable-tls: "true"

spec:
  ingressClassName: avi-lb
  rules:
    - host: fruit-site-2.lab.guzware.net
      http:
        paths:
        - path: /apple
          pathType: Prefix
          backend:
            service:
              name: apple-service
              port:
                number: 5678
        - path: /banana
          pathType: Prefix
          backend:
            service:
              name: banana-service
              port:
                number: 5678
---
apiVersion: ako.vmware.com/v1alpha1
kind: HostRule
metadata:
  namespace: fruit
  name: gslb-host-rule-fruit
spec:
  virtualhost:
    fqdn: fruit-site-2.lab.guzware.net
    enableVirtualHost: true
    gslb:
      fqdn: fruit.gslb.guzware.net
```
