1. Generate a new RSA key:
    
    ```
    openssl genrsa -out nitish.key 2048
    
    ```
    
2. Create a new certificate signing request:
    
    ```
    openssl req -new -key nitish.key -out nitish.csr -subj "/CN=nitish/0=group1”
    
    ```
    
3. Encode the CSR in Base64:
    
    ```
    cat nitish.csr | base64 | tr -d '\\n’
    
    ```
    
4. Paste the output into a CSR configuration file (csr.yaml):
    
    ```
    apiVersion: certificates.k8s.io/v1
    kind: CertificateSigningRequest
    metadata:
      name: saiyam
    spec:
      request: BASE64_CSR
      signerName: kubernetes.io/kube-apiserver-client
      usages:
      - client auth
    
    ```
    
5. Apply the CSR:
    
    ```
    kubectl apply -f csr.yaml
    
    ```
    
6. Approve the certificate:
    
    ```
    kubectl certificate approve nitish
    
    ```
    
7. Get the certificate:
    
    ```
    kubectl get csr nitish -o jsonpath='{.status.certificate}' | base64 --decode > nitish.crt
    
    ```
    
8. Create a Role and a RoleBinding in a configuration file (role.yaml):
    
    ```
    kind: Role
    apiVersion: rbac.authorization.k8s.io/v1
    metadata:
      namespace: default
      name: pod-reader
    rules:
    - apiGroups: [""]
      resources: ["pods"]
      verbs: ["get", "watch", "list"]
    
    kind: RoleBinding
    apiVersion: rbac.authorization.k8s.io/v1
    metadata:
      name: read-pods
      namespace: default
    subjects:
    - kind: User
      name: nitish
      apiGroup: rbac.authorization.k8s.io
    roleRef:
      kind: Role
      name: pod-reader
      apiGroup: rbac.authorization.k8s.io
    
    ```
    
9. Set up kubeconfig:
    
    ```
    kubectl config set-credentials saiyam --client-certificate=saiyam.crt --client-key=saiyam.key
    kubectl config set-context nitish-context --cluster=kubernetes --namespace=default --user=nitish
    
    ```
