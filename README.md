# DsitributedTrackingApplications

apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: mysql-pv-claim
  labels:
    app: mysql
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 1Gi
---
apiVersion: v1
kind: Service
metadata:
  name: mysql-service
spec:
  selector:
    app: mysql
  ports:
    - protocol: TCP
      port: 3306
      targetPort: 3306
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: mysql-deployment
spec:
  replicas: 1
  selector:
    matchLabels:
      app: mysql
  template:
    metadata:
      labels:
        app: mysql
    spec:
      containers:
        - name: mysql
          image: mysql:5.7
          env:
            - name: MYSQL_ROOT_PASSWORD
              value: your_password_here
          ports:
            - containerPort: 3306
          volumeMounts:
            - mountPath: /var/lib/mysql
              name: mysql-persistent-storage
      volumes:
        - name: mysql-persistent-storage
          persistentVolumeClaim:
            claimName: mysql-pv-claim
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: openzipkin
spec:
  replicas: 1
  selector:
    matchLabels:
      app: openzipkin
  template:
    metadata:
      labels:
        app: openzipkin
    spec:
      containers:
        - name: openzipkin
          image: openzipkin/zipkin:2.24.4
          ports:
            - containerPort: 9411
          env:
            - name: STORAGE_TYPE
              value: mysql
            - name: MYSQL_HOST
              value: mysql-service
            - name: MYSQL_DATABASE
              value: zipkin
            - name: MYSQL_USER
              value: your_mysql_user
            - name: MYSQL_PASSWORD
              value: your_mysql_password
---
apiVersion: v1
kind: Service
metadata:
  name: openzipkin-service
spec:
  selector:
    app: openzipkin
  ports:
    - protocol: TCP
      port: 9411
      targetPort: 9411
