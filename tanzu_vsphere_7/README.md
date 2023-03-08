# Tanzu with vSphere 7
Here I have collected the yaml definitions I am using regularly for Tanzu with vSphere 7.

## ClusterRole 
```
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRole
metadata:
  name: psp:privileged
rules:
- apiGroups: ['policy']
  resources: ['podsecuritypolicies']
  verbs:     ['use']
  resourceNames:
  - vmware-system-privileged
---
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  name: all:psp:privileged
roleRef:
  kind: ClusterRole
  name: psp:privileged
  apiGroup: rbac.authorization.k8s.io
subjects:
- kind: Group
  name: system:serviceaccounts
  apiGroup: rbac.authorization.k8s.io
```
## Tanzu TKC cluster
```
apiVersion: run.tanzu.vmware.com/v1alpha2
kind: TanzuKubernetesCluster
metadata:
  name: tkc-cluster-1
  namespace: dc-ns-1
spec:
  topology:
    controlPlane:
      replicas: 3
      vmClass: best-effort-medium
      storageClass: tanzu-default-storage
      tkr:  
        reference:
          name: v1.23.8---vmware.3-tkg.1
    nodePools:
    - name: pool-1
      replicas: 3
      vmClass: best-effort-medium
      storageClass: tanzu-default-storage
      tkr:  
        reference:
          name: v1.23.8---vmware.3-tkg.1
  settings:
    storage:
      defaultClass: tanzu-default-storage
    network:
      cni:
        name: antrea
      services:
        cidrBlocks: ["20.10.0.0/16"]
      pods:
        cidrBlocks: ["20.20.0.0/16"]
      serviceDomain: cluster.local
#      proxy:
#        httpProxy: http://proxy.server.com:8080
#        httpsProxy: http://proxy.server.com:8080
#        noProxy: [10.96.0.0/12,192.168.0.0/16,"mgmt-frontend-workload-network-cidr",.local,.svc,.svc.cluster.local]
#      trust:
#        additionalTrustedCAs:
#          - name: CompanyInternalCA-1
#            data: LS0tLS1C...LS0tCg==
#          - name: CompanyInternalCA-2
#            data: MTLtMT1C...MT0tPg==
```

