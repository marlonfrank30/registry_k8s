apiVersion: apps/v1
kind: Deployment
metadata:
  name: registry
  labels:
    app: registry
spec:
  replicas: 1
  selector:
    matchLabels:
      app: registry
  template:
    metadata:
      labels:
        app: registry
    spec:
      containers:
      - name: registry
        env:
#        - name: REGISTRY_AUTH
#          value: htpasswd 
#        - name: REGISTRY_AUTH_HTPASSWD_REALM
#          value: "Registry Realm"
#        - name: REGISTRY_AUTH_HTPASSWD_PATH
#          value: /auth/htpasswd
        - name: REGISTRY_HTTP_ADDR
          value: 0.0.0.0:5000
        - name: REGISTRY_HTTP_TLS_CERTIFICATE
          value: /certs/public/domain.crt
        - name: REGISTRY_HTTP_TLS_KEY
          value: /certs/private/domain.key
        - name: REGISTRY_HTTP_HEADERS_Access-Control-Allow-Origin
          value: "[\"*\"]"
        image: registry:2.7.0
        imagePullPolicy: Always
        ports:
        - containerPort: 5000
          name: registry
          protocol: TCP
        volumeMounts:
        - mountPath: /var/lib/registry
          name: registry-repository
        - mountPath: /certs
          name: registry-certs
        - mountPath: /etc/docker/registry/config.yml
          name: registry-config
        - mountPath: /auth
          name: registry-auth
      volumes:
      - name: registry-repository
        hostPath:
            path: /kubefilesystem/appfiles/registry
            type: Directory
      - name: registry-auth
        hostPath:
            path: /kubefilesystem/appfiles/registry/auth
            type: Directory
      - name: registry-certs
        hostPath:
            path: /kubefilesystem/appfiles/registry/certs
            type: Directory
      - name: registry-config
        hostPath:
            path: /kubefilesystem/appfiles/registry/config.yml
            type: File
---
apiVersion: v1
kind: Service
metadata:
  name: registery-vs
  annotations:
    metallb.universe.tf/allow-shared-ip: default
spec:
  ports:
  - port: 5000
    name: registry
    targetPort: 5000
    protocol: TCP
  loadBalancerIP: 192.168.0.30
  selector:
    app: registry
  type: LoadBalancer
