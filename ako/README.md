# AKO section
In here I have collected all the YAML definitions I am using regularly related to Avi and AKO like Ingress, ServiceType LoadBlancer, AKO CRDs etc

## AKO default values.yaml
```
# Default values for ako.
# This is a YAML-formatted file.
# Declare variables to be passed into your templates.

replicaCount: 1

image:
  repository: projects.registry.vmware.com/ako/ako
  pullPolicy: IfNotPresent

### This section outlines the generic AKO settings
AKOSettings:
  primaryInstance: true # Defines AKO instance is primary or not. Value `true` indicates that AKO instance is primary. In a multiple AKO deployment in a cluster, only one AKO instance should be primary. Default value: true.
  enableEvents: 'true' # Enables/disables Event broadcasting via AKO 
  logLevel: WARN   # enum: INFO|DEBUG|WARN|ERROR
  fullSyncFrequency: '1800' # This frequency controls how often AKO polls the Avi controller to update itself with cloud configurations.
  apiServerPort: 8080 # Internal port for AKO's API server for the liveness probe of the AKO pod default=8080
  deleteConfig: 'false' # Has to be set to true in configmap if user wants to delete AKO created objects from AVI 
  disableStaticRouteSync: 'false' # If the POD networks are reachable from the Avi SE, set this knob to true.
  clusterName: my-cluster   # A unique identifier for the kubernetes cluster, that helps distinguish the objects for this cluster in the avi controller. // MUST-EDIT
  cniPlugin: '' # Set the string if your CNI is calico or openshift. enum: calico|canal|flannel|openshift|antrea|ncp
  enableEVH: false # This enables the Enhanced Virtual Hosting Model in Avi Controller for the Virtual Services
  layer7Only: false # If this flag is switched on, then AKO will only do layer 7 loadbalancing.
  # NamespaceSelector contains label key and value used for namespacemigration
  # Same label has to be present on namespace/s which needs migration/sync to AKO
  namespaceSelector:
    labelKey: ''
    labelValue: ''
  servicesAPI: false # Flag that enables AKO in services API mode: https://kubernetes-sigs.github.io/service-apis/. Currently implemented only for L4. This flag uses the upstream GA APIs which are not backward compatible 
                     # with the advancedL4 APIs which uses a fork and a version of v1alpha1pre1 
  vipPerNamespace: 'false' # Enabling this flag would tell AKO to create Parent VS per Namespace in EVH mode
  istioEnabled: false # This flag needs to be enabled when AKO is be to brought up in an Istio environment
  # This is the list of system namespaces from which AKO will not listen any Kubernetes or Openshift object event.
  blockedNamespaceList: []
  # blockedNamespaceList:
  #   - kube-system
  #   - kube-public
  ipFamily: '' # This flag can take values V4 or V6 (default V4). This is for the backend pools to use ipv6 or ipv4. For frontside VS, use v6cidr


### This section outlines the network settings for virtualservices. 
NetworkSettings:
  ## This list of network and cidrs are used in pool placement network for vcenter cloud.
  ## Node Network details are not needed when in nodeport mode / static routes are disabled / non vcenter clouds.
  nodeNetworkList: []
  # nodeNetworkList:
  #   - networkName: "network-name"
  #     cidrs:
  #       - 10.0.0.1/24
  #       - 11.0.0.1/24
  enableRHI: false # This is a cluster wide setting for BGP peering.
  nsxtT1LR: '' # T1 Logical Segment mapping for backend network. Only applies to NSX-T cloud.
  bgpPeerLabels: [] # Select BGP peers using bgpPeerLabels, for selective VsVip advertisement.
  # bgpPeerLabels:
  #   - peer1
  #   - peer2
  vipNetworkList: [] # Network information of the VIP network. Multiple networks allowed only for AWS Cloud.
  # vipNetworkList:
  #  - networkName: net1
  #    cidr: 100.1.1.0/24
  #    v6cidr: 2002::1234:abcd:ffff:c0a8:101/64 # Setting this will enable the VS networks to use ipv6 

### This section outlines all the knobs  used to control Layer 7 loadbalancing settings in AKO.
L7Settings:
  defaultIngController: 'true'
  noPGForSNI: false # Switching this knob to true, will get rid of poolgroups from SNI VSes. Do not use this flag, if you don't want http caching. This will be deprecated once the controller support caching on PGs.
  serviceType: ClusterIP # enum NodePort|ClusterIP|NodePortLocal
  shardVSSize: LARGE   # Use this to control the layer 7 VS numbers. This applies to both secure/insecure VSes but does not apply for passthrough. ENUMs: LARGE, MEDIUM, SMALL, DEDICATED
  passthroughShardSize: SMALL   # Control the passthrough virtualservice numbers using this ENUM. ENUMs: LARGE, MEDIUM, SMALL
  enableMCI: 'false' # Enabling this flag would tell AKO to start processing multi-cluster ingress objects.

### This section outlines all the knobs  used to control Layer 4 loadbalancing settings in AKO.
L4Settings:
  defaultDomain: '' # If multiple sub-domains are configured in the cloud, use this knob to set the default sub-domain to use for L4 VSes.
  autoFQDN: default   # ENUM: default(<svc>.<ns>.<subdomain>), flat (<svc>-<ns>.<subdomain>), "disabled" If the value is disabled then the FQDN generation is disabled.

### This section outlines settings on the Avi controller that affects AKO's functionality.
ControllerSettings:
  serviceEngineGroupName: Default-Group   # Name of the ServiceEngine Group.
  controllerVersion: '' # The controller API version
  cloudName: Default-Cloud   # The configured cloud name on the Avi controller.
  controllerHost: '' # IP address or Hostname of Avi Controller
  tenantName: admin   # Name of the tenant where all the AKO objects will be created in AVI.

nodePortSelector: # Only applicable if serviceType is NodePort
  key: ''
  value: ''

resources:
  limits:
    cpu: 350m
    memory: 400Mi
  requests:
    cpu: 200m
    memory: 300Mi

securityContext: {}

podSecurityContext: {}

rbac:
  # Creates the pod security policy if set to true
  pspEnable: false


avicredentials:
  username: ''
  password: ''
  authtoken:
  certificateAuthorityData:


persistentVolumeClaim: ''
mountPath: /log
logFile: avi.log
```

## AviInfraSetting - enable BGP (override default settings)
```
apiVersion: ako.vmware.com/v1alpha1
kind: AviInfraSetting
metadata:
  name: enable-bgp-fruit
spec:
  seGroup:
    name: Default-Group
  network:
    vipNetworks:
      - networkName: vds-tkc-frontend-l7-vlan-1028
        cidr: 10.102.8.0/24
    nodeNetworks:
      - networkName: vds-tkc-workload-vlan-1026
        cidrs:
        - 10.102.6.0/24
    enableRhi: true
    bgpPeerLabels:
      - cPodRouter
```

## AKO Ingress Class using AviInfraSetting
```
apiVersion: networking.k8s.io/v1
kind: IngressClass
metadata:
  name: avi-lb-bgp
spec:
  controller: ako.vmware.com/avi-lb
  parameters:
    apiGroup: ako.vmware.com
    kind: AviInfraSetting
    name: enable-bgp-fruit
```

## Ingress example with custom ingress class to enable BGP
```
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: ingress-example
  namespace: fruit
#  annotations:
#    aviinfrasetting.ako.vmware.com/name: "enable-bgp-fruit"
#    ako.vmware.com/enable-tls: "true"

spec:
  ingressClassName: avi-lb-bgp
  rules:
    - host: fruit-tkgs.you-have.your-domain.here
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
```

## App "apple" used to test Ingress
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
        - "-text=apple"

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
## App "banana" used to test Ingress
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
        - "-text=banana"

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
### App "apple" to test Ingress with NodePort
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
        - "-text=apple"

---

kind: Service
apiVersion: v1
metadata:
  name: apple-service
  namespace: fruit
spec:
  type: NodePort
  selector:
    app: apple
  ports:
    - port: 5678 # Default port for image
```

### App "banana" to test Ingress with NodePort
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
        - "-text=banana"

---

kind: Service
apiVersion: v1
metadata:
  name: banana-service
  namespace: fruit
spec:
  type: NodePort
  selector:
    app: banana
  ports:
    - port: 5678 # Default port for image
```
## AKO HTTPRule - Persistence 
```
apiVersion: ako.vmware.com/v1alpha1
kind: HTTPRule
metadata:
   name: fruit-httprule
   namespace: fruit
spec:
  fqdn: fruit-tkgs.carefor.some-dns.net
  paths:
  - target: /
    loadBalancerPolicy:
      algorithm: LB_ALGORITHM_ROUND_ROBIN
    healthMonitors:
      - System-HTTP
    applicationPersistence: System-Persistence-Http-Cookie
```
## AKO HTTPRule - Persistence - consistent hash
```
apiVersion: ako.vmware.com/v1alpha1
kind: HTTPRule
metadata:
   name: httprule-fruit
   namespace: fruit
spec:
  fqdn: fruit-tkgm.carefor.some-dns.net
  paths:
  - target: /
    loadBalancerPolicy:
      algorithm: LB_ALGORITHM_CONSISTENT_HASH
      hash: LB_ALGORITHM_CONSISTENT_HASH_SOURCE_IP_ADDRESS
    healthMonitors:
      - System-HTTP
    applicationPersistence: System-Persistence-Http-Cookie
```
## AKO HostRule - SSL termination and SSL Certificates
```
apiVersion: ako.vmware.com/v1alpha1
kind: HostRule
metadata:
  name: my-ingress-example-http-rule
  namespace: fruit
spec:
  virtualhost:
    fqdn: fruit.guzware.net # mandatory
    fqdnType: Exact
    enableVirtualHost: true
    tls: # optional
      sslKeyCertificate:
        name: "guzware ECDSA"
        type: ref
        alternateCertificate:
          name: "Guzware RSA"
          type: ref
      sslProfile: System-Standard-PFS
      termination: edge
```
## AKO Default route certificate
```
apiVersion: v1
kind: Secret
metadata:
  name: router-certs-default
  namespace: avi-system
type: kubernetes.io/tls
data:
  tls.crt: base64 - RSA
  tls.key: base64 - RSA
  alt.crt: base64 - ECDSA
  alt.key: base64 - ECDSA
```
## AKO Hostrule trying to force TLS on SharedL7 object - not working I think...
```
apiVersion: ako.vmware.com/v1alpha1
kind: HostRule
metadata:
  name: avi-host-rule
  namespace: avi-system
spec:
  virtualhost:
    fqdn: 'tkgs-v1-cluster--Shared-L7-0.admin.avi.andreasmlab.net' # mandatory
    fqdnType: Contains
    enableVirtualHost: true
    tls: # optional
      sslKeyCertificate:
        name: "wildcard-certificate"
        type: ref
      sslProfile: System-Standard-PFS
      termination: edge
```
## AKO HTTPRule - Re-Encrypt
```
apiVersion: ako.vmware.com/v1alpha1
kind: HTTPRule
metadata:
   name: unifi-http-rule
   namespace: unifi
spec:
  fqdn: unifi-mgmt.guzware.net
  paths:
  - target: /
    tls: ## This is a re-encrypt to pool
      type: reencrypt # Mandatory [re-encrypt]
      sslProfile: System-Standard-PFS
      #pkiProfile: 
```
## AKO ServiceType LoadBalancer - multi ports - TCP/UDP - same VIP
```
kind: Service
apiVersion: v1
metadata:
  name: lb-unifi-tcp
  namespace: unifi
  annotations:
    external-dns.alpha.kubernetes.io/hostname: unifi.int.guzware.net
#    ako.vmware.com/enable-shared-vip: "shared-vip-key-1"
spec:
  ports:
    - name: device-comm
      protocol: TCP
      port: 8080
    - name: default-console
      protocol: TCP
      port: 8443
    - name: secure-redirect
      protocol: TCP
      port: 8843
    - name: http-redirect
      protocol: TCP
      port: 8880
#    - name: speedtest
#      protocol: TCP
#      port: 6789
    - name: stun
      protocol: UDP
      port: 3478
    - name: unifi-disc
      port: 10001
      protocol: UDP
#    - name: unifi-disc-12
#      port: 1900
#      protocol: UDP
  selector:
    name: unifi-controller
  type: LoadBalancer
  loadBalancerIP: 10.150.11.199
#---
#kind: Service
#apiVersion: v1
#metadata:
#  name: lb-unifi-udp
#  namespace: unifi
#  annotations:
#    ako.vmware.com/enable-shared-vip: "shared-vip-key-1"
#spec:
#  ports:
#    - name: stun
#      protocol: UDP
#      port: 3478
#    - name: unifi-disc
#      port: 10001
#      protocol: UDP
#    - name: unifi-disc-12
#      port: 1900
#      protocol: UDP
#  selector:
#    name: unifi-controller
#  type: LoadBalancer
#  loadBalancerIP: 10.150.11.199
```
## AKO HostRule - with TCP listeners
```
apiVersion: ako.vmware.com/v1alpha1
kind: HostRule
metadata:
  name: pinniped-http-rule
  namespace: pinniped-supervisor
spec:
  virtualhost:
    fqdn: pinniped.guzware.net # mandatory
    fqdnType: Exact
    enableVirtualHost: true
    tls: # optional
      sslKeyCertificate:
        name: "pinniped-cert"
        type: ref
      sslProfile: System-Standard-PFS
      termination: edge
    applicationProfile: app-profile-pinniped
    tcpSettings:
      listeners:
      - port: 443
        enableSSL: true
```
## AKO HTTPRule - Re-Encrypt
```
apiVersion: ako.vmware.com/v1alpha1
kind: HTTPRule
metadata:
   name: ako-pinnoiped-http-rule
   namespace: pinniped-supervisor
spec:
  fqdn: pinniped.guzware.net
  paths:
  - target: /
    healthMonitors:
    - pinniped-https
    tls: ## This is a re-encrypt to pool
      type: reencrypt # Mandatory [re-encrypt]
      sslProfile: System-Standard-PFS
      destinationCA:  |-
        -----BEGIN CERTIFICATE-----
        -----END CERTIFICATE-----
```
                                                                                                                                                                                              
## AKO HTTPRule Health Monitor
```
apiVersion: ako.vmware.com/v1alpha1
kind: HTTPRule
metadata:
   name: ako-pinnoiped-http-rule
   namespace: pinniped-supervisor
spec:
  fqdn: pinniped.guzware.net
  paths:
  - target: /
    healthMonitors:
    - pinniped-https
```

