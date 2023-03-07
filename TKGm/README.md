# TKGm section
In this section I have collected all the relevant yaml defintions I have been using for TKGm

## TKG Management Cluster bootstrap yaml
```
#! ---------------
#! Basic config
#! -------------
CLUSTER_NAME: tkg-stc-mgmt-cluster
CLUSTER_PLAN: dev
INFRASTRUCTURE_PROVIDER: vsphere
ENABLE_CEIP_PARTICIPATION: "false"
ENABLE_AUDIT_LOGGING: "false"
CLUSTER_CIDR: 100.96.0.0/11
SERVICE_CIDR: 100.64.0.0/13
TKG_IP_FAMILY: ipv4
DEPLOY_TKG_ON_VSPHERE7: "true"

#! ---------------
#! vSphere config
#! -------------
VSPHERE_DATACENTER: /cPod-NSXAM-STC
VSPHERE_DATASTORE: /cPod-NSXAM-STC/datastore/vsanDatastore
VSPHERE_FOLDER: /cPod-NSXAM-STC/vm/TKGm
VSPHERE_INSECURE: "false"
VSPHERE_NETWORK: /cPod-NSXAM-STC/network/ls-tkg-mgmt
VSPHERE_CONTROL_PLANE_ENDPOINT: ""
VSPHERE_PASSWORD: ""
VSPHERE_RESOURCE_POOL: /cPod-NSXAM-STC/host/Cluster/Resources
#VSPHERE_TEMPLATE: /Datacenter/vm/TKGm/ubuntu-2004-kube-v1.23.8+vmware.2
VSPHERE_SERVER: vcsa.cpod-nsxam-stc.az-stc.cloud-garage.net
VSPHERE_SSH_AUTHORIZED_KEY: ssh-rsa AAbw1Q== andreasm@hisworkstation 
VSPHERE_TLS_THUMBPRINT: 22:DF
VSPHERE_USERNAME: andreasm@cpod-nsxam-stc.az-stc.cloud-garage.net

#! ---------------
#! Node config
#! -------------
OS_ARCH: amd64
OS_NAME: ubuntu
OS_VERSION: "20.04"
VSPHERE_CONTROL_PLANE_DISK_GIB: "20"
VSPHERE_CONTROL_PLANE_MEM_MIB: "4096"
VSPHERE_CONTROL_PLANE_NUM_CPUS: "2"
VSPHERE_WORKER_DISK_GIB: "20"
VSPHERE_WORKER_MEM_MIB: "4096"
VSPHERE_WORKER_NUM_CPUS: "2"
CONTROL_PLANE_MACHINE_COUNT: 1
WORKER_MACHINE_COUNT: 2

#! ---------------
#! Avi config
#! -------------
AVI_CA_DATA_B64: LS0
AVI_CLOUD_NAME: stc-nsx-cloud
AVI_CONTROL_PLANE_HA_PROVIDER: "true"
AVI_CONTROLLER: 172.24.3.50
# Network used to place workload clusters' endpoint VIPs
AVI_CONTROL_PLANE_NETWORK: vip-tkg-wld-l4
AVI_CONTROL_PLANE_NETWORK_CIDR: 10.13.102.0/24
# Network used to place workload clusters' services external IPs (load balancer & ingress services)
AVI_DATA_NETWORK: vip-tkg-wld-l7
AVI_DATA_NETWORK_CIDR: 10.13.103.0/24
# Network used to place management clusters' services external IPs (load balancer & ingress services)
AVI_MANAGEMENT_CLUSTER_VIP_NETWORK_CIDR: 10.13.101.0/24
AVI_MANAGEMENT_CLUSTER_VIP_NETWORK_NAME: vip-tkg-mgmt-l7
# Network used to place management clusters' endpoint VIPs
AVI_MANAGEMENT_CLUSTER_CONTROL_PLANE_VIP_NETWORK_NAME: vip-tkg-mgmt-l4
AVI_MANAGEMENT_CLUSTER_CONTROL_PLANE_VIP_NETWORK_CIDR: 10.13.100.0/24
AVI_NSXT_T1LR: Tier-1
AVI_CONTROLLER_VERSION: 21.1.4
AVI_ENABLE: "true"
AVI_LABELS: ""
AVI_PASSWORD: ""
AVI_SERVICE_ENGINE_GROUP: stc-nsx
# AVI_MANAGEMENT_CLUSTER_SERVICE_ENGINE_GROUP: tkgm-se-group
AVI_USERNAME: admin
# AVI_NAMESPACE: "tkg-system-networking"
# AVI_DISABLE_INGRESS_CLASS: false
# AVI_AKO_IMAGE_PULL_POLICY: IfNotPresent
# AVI_ADMIN_CREDENTIAL_NAME: avi-controller-credentials
# AVI_CA_NAME: avi-controller-ca
AVI_MANAGEMENT_CLUSTER_SERVICE_ENGINE_GROUP: stc-nsx
AVI_DISABLE_STATIC_ROUTE_SYNC: true
AVI_INGRESS_DEFAULT_INGRESS_CONTROLLER: false
AVI_INGRESS_SHARD_VS_SIZE: SMALL
AVI_INGRESS_SERVICE_TYPE: NodePortLocal
AVI_DISABLE_INGRESS_CLASS: false
AVI_CNI_PLUGIN: antrea
# AVI_INGRESS_NODE_NETWORK_LIST: ""

#! ---------------
#! Proxy config
#! -------------
TKG_HTTP_PROXY_ENABLED: "false"

#! ---------------------------------------------------------------------
#! Antrea CNI configuration
#! ---------------------------------------------------------------------
# ANTREA_NO_SNAT: false
# ANTREA_TRAFFIC_ENCAP_MODE: "encap"
# ANTREA_PROXY: false
# ANTREA_POLICY: true
# ANTREA_TRACEFLOW: false
ANTREA_NODEPORTLOCAL: true
ANTREA_PROXY: true
ANTREA_ENDPOINTSLICE: true
ANTREA_POLICY: true
ANTREA_TRACEFLOW: true
ANTREA_NETWORKPOLICY_STATS: false
ANTREA_EGRESS: true
ANTREA_IPAM: false
ANTREA_FLOWEXPORTER: false
ANTREA_SERVICE_EXTERNALIP: false
ANTREA_MULTICAST: false

#! ---------------------------------------------------------------------
#! Machine Health Check configuration
#! ---------------------------------------------------------------------
ENABLE_MHC: "true"
ENABLE_MHC_CONTROL_PLANE: true
ENABLE_MHC_WORKER_NODE: true
MHC_UNKNOWN_STATUS_TIMEOUT: 5m
MHC_FALSE_STATUS_TIMEOUT: 12m

#! ---------------------------------------------------------------------
#! Identity management configuration
#! ---------------------------------------------------------------------

IDENTITY_MANAGEMENT_TYPE: none
#LDAP_BIND_DN: CN=Andreas M,OU=Users,OU=GUZWARE,DC=guzware,DC=local
#LDAP_BIND_PASSWORD: <encoded:UHNAc=>
#LDAP_GROUP_SEARCH_BASE_DN: DC=guzware,DC=local
#LDAP_GROUP_SEARCH_FILTER: (objectClass=group)
#LDAP_GROUP_SEARCH_GROUP_ATTRIBUTE: member
#LDAP_GROUP_SEARCH_NAME_ATTRIBUTE: cn
#LDAP_GROUP_SEARCH_USER_ATTRIBUTE: distinguishedName
#LDAP_HOST: guzad07.guzware.local:636
#LDAP_ROOT_CA_DATA_B64: LS0tLS1CRUd
#LDAP_USER_SEARCH_BASE_DN: DC=guzware,DC=local
#LDAP_USER_SEARCH_FILTER: (objectClass=person)
#LDAP_USER_SEARCH_NAME_ATTRIBUTE: uid
#LDAP_USER_SEARCH_USERNAME: uid
#OIDC_IDENTITY_PROVIDER_CLIENT_ID: ""
#OIDC_IDENTITY_PROVIDER_CLIENT_SECRET: ""
#OIDC_IDENTITY_PROVIDER_GROUPS_CLAIM: ""
#OIDC_IDENTITY_PROVIDER_ISSUER_URL: ""
#OIDC_IDENTITY_PROVIDER_NAME: ""
#OIDC_IDENTITY_PROVIDER_SCOPES: ""
#OIDC_IDENTITY_PROVIDER_USERNAME_CLAIM: ""
```
## TKG Workload bootstrap
```
#! -------
#! Basic cluster creation config
#! --------

CLUSTER_NAME: tkg-stc-wld-cluster-1
CLUSTER_PLAN: dev
NAMESPACE: ns-stc-tkg-wld-1
INFRASTRUCTURE_PROVIDER: vsphere
IDENTITY_MANAGEMENT_TYPE: none
TKG_IP_FAMILY: ipv4


#! -----
#! Node Config
#! ---

# SIZE: small
# CONTROLPLANE_SIZE:
# WORKER_SIZE:

# VSPHERE_NUM_CPUS: 2
# VSPHERE_DISK_GIB: 40
# VSPHERE_MEM_MIB: 4096

VSPHERE_CONTROL_PLANE_NUM_CPUS: 4
VSPHERE_CONTROL_PLANE_DISK_GIB: 20
VSPHERE_CONTROL_PLANE_MEM_MIB: 6144
VSPHERE_WORKER_NUM_CPUS: 4
VSPHERE_WORKER_DISK_GIB: 40
VSPHERE_WORKER_MEM_MIB: 8192

CONTROL_PLANE_MACHINE_COUNT: 1
WORKER_MACHINE_COUNT: 3
# WORKER_MACHINE_COUNT_0:
# WORKER_MACHINE_COUNT_1:
# WORKER_MACHINE_COUNT_2:


AVI_CONTROL_PLANE_HA_PROVIDER: "true"

#! ---------------
#! vSphere config
#! -------------
VSPHERE_DATACENTER: /cPod-NSXAM-STC
VSPHERE_DATASTORE: /cPod-NSXAM-STC/datastore/vsanDatastore
VSPHERE_FOLDER: /cPod-NSXAM-STC/vm/TKGm
VSPHERE_INSECURE: "false"
VSPHERE_NETWORK: /cPod-NSXAM-STC/network/ls-tkg-wld-1
VSPHERE_CONTROL_PLANE_ENDPOINT: ""
VSPHERE_PASSWORD: ""
VSPHERE_RESOURCE_POOL: /cPod-NSXAM-STC/host/Cluster/Resources
#VSPHERE_TEMPLATE: /Datacenter/vm/TKGm/ubuntu-2004-kube-v1.23.8+vmware.2
VSPHERE_SERVER: vcsa.cpod-nsxam-stc.az-stc.cloud-garage.net
VSPHERE_SSH_AUTHORIZED_KEY: ssh-rsa gb== andreasm@hisworkstation
VSPHERE_TLS_THUMBPRINT: 22:DF
VSPHERE_USERNAME: andreasm@cpod-nsxam-stc.az-stc.cloud-garage.net





#! ------
#! Machine Health Check Config
#! -------
ENABLE_MHC:
ENABLE_MHC_CONTROL_PLANE: true
ENABLE_MHC_WORKER_NODE: true
MHC_UNKNOWN_STATUS_TIMEOUT: 5m
MHC_FALSE_STATUS_TIMEOUT: 12m

#! -----
#! Common config
#! -------

ENABLE_AUDIT_LOGGING: "false"

CLUSTER_CIDR: 100.96.0.0/11
SERVICE_CIDR: 100.64.0.0/13
TKG_HTTP_PROXY_ENABLED: "false"
TKG_IP_FAMILY: ipv4
OS_ARCH: amd64
OS_NAME: ubuntu
OS_VERSION: "20.04"

#! ---------------------------------------------------------------------
#! Autoscaler configuration
#! ---------------------------------------------------------------------

ENABLE_AUTOSCALER: false
# AUTOSCALER_MAX_NODES_TOTAL: "0"
# AUTOSCALER_SCALE_DOWN_DELAY_AFTER_ADD: "10m"
# AUTOSCALER_SCALE_DOWN_DELAY_AFTER_DELETE: "10s"
# AUTOSCALER_SCALE_DOWN_DELAY_AFTER_FAILURE: "3m"
# AUTOSCALER_SCALE_DOWN_UNNEEDED_TIME: "10m"
# AUTOSCALER_MAX_NODE_PROVISION_TIME: "15m"
# AUTOSCALER_MIN_SIZE_0:
# AUTOSCALER_MAX_SIZE_0:
# AUTOSCALER_MIN_SIZE_1:
# AUTOSCALER_MAX_SIZE_1:
# AUTOSCALER_MIN_SIZE_2:
# AUTOSCALER_MAX_SIZE_2:

#! ---------------------------------------------------------------------
#! Antrea CNI configuration
#! ---------------------------------------------------------------------
# ANTREA_NO_SNAT: false
# ANTREA_TRAFFIC_ENCAP_MODE: "encap"
# ANTREA_PROXY: false
# ANTREA_POLICY: true
# ANTREA_TRACEFLOW: false
ANTREA_NODEPORTLOCAL: true
```
