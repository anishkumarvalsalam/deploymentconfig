# deploymentconfig

kind: DeploymentConfig
apiVersion: apps.openshift.io/v1
metadata:
  name: broker-amq
  namespace: dev-project
spec:
  strategy:
    type: Rolling
    rollingParams:
      updatePeriodSeconds: 1
      intervalSeconds: 1
      timeoutSeconds: 600
      maxUnavailable: 25%
      maxSurge: 0
    resources: {}
    activeDeadlineSeconds: 21600
  triggers:
    - type: ImageChange
      imageChangeParams:
        automatic: true
        containerNames:
          - broker-amq
        from:
          kind: ImageStreamTag
          namespace: openshift
          name: 'jboss-amq-63:1.4'
        lastTriggeredImage: >-
          image-registry.openshift-image-registry.svc:5000/openshift/jboss-amq-63
    - type: ConfigChange
  replicas: 1
  revisionHistoryLimit: 10
  test: false
  selector:
    deploymentConfig: broker-amq
  template:
    metadata:
      name: broker-amq
      creationTimestamp: null
      labels:
        application: broker
        deploymentConfig: broker-amq
    spec:
      volumes:
        - name: broker-amq-pvol
          persistentVolumeClaim:
            claimName: broker-amq-claim
      containers:
        - resources: {}
          readinessProbe:
            exec:
              command:
                - /bin/bash
                - '-c'
                - /opt/amq/bin/readinessProbe.sh
            timeoutSeconds: 1
            periodSeconds: 10
            successThreshold: 1
            failureThreshold: 3
          terminationMessagePath: /dev/termination-log
          name: broker-amq
          env:
            - name: AMQ_USER
              value: admin
            - name: AMQ_PASSWORD
              value: admin
            - name: AMQ_TRANSPORTS
              value: openwire
            - name: AMQ_QUEUES
            - name: AMQ_TOPICS
            - name: MQ_SERIALIZABLE_PACKAGES
            - name: AMQ_SPLIT
              value: 'true'
            - name: AMQ_MESH_DISCOVERY_TYPE
              value: dns
            - name: AMQ_MESH_SERVICE_NAME
              value: broker-amq-mesh
            - name: AMQ_MESH_SERVICE_NAMESPACE
              valueFrom:
                fieldRef:
                  apiVersion: v1
                  fieldPath: metadata.namespace
            - name: AMQ_STORAGE_USAGE_LIMIT
              value: 1 gb
            - name: AMQ_QUEUE_MEMORY_LIMIT
            - name: AMQ_CONF
              valueFrom:
                configMapKeyRef:
                  name: amqconfig
                  key: activemq.xml
          ports:
            - name: jolokia
              containerPort: 8778
              protocol: TCP
            - name: amqp
              containerPort: 5672
              protocol: TCP
            - name: mqtt
              containerPort: 1883
              protocol: TCP
            - name: stomp
              containerPort: 61613
              protocol: TCP
            - name: tcp
              containerPort: 61616
              protocol: TCP
          imagePullPolicy: Always
          volumeMounts:
            - name: broker-amq-pvol
              mountPath: /opt/amq/data
          terminationMessagePolicy: File
          image: >-
            image-registry.openshift-image-registry.svc:5000/openshift/jboss-amq-63
      restartPolicy: Always
      terminationGracePeriodSeconds: 60
      dnsPolicy: ClusterFirst
      securityContext: {}
      schedulerName: default-scheduler
status:
  observedGeneration: 21
  details:
    message: config change
    causes:
      - type: ConfigChange
  availableReplicas: 1
  unavailableReplicas: 0
  latestVersion: 5
  updatedReplicas: 1
  conditions:
    - type: Progressing
      status: 'True'
      lastUpdateTime: '2022-03-29T01:05:38Z'
      lastTransitionTime: '2022-03-29T01:05:38Z'
      reason: NewReplicationControllerAvailable
      message: replication controller "broker-amq-5" successfully rolled out
    - type: Available
      status: 'True'
      lastUpdateTime: '2022-03-29T01:07:40Z'
      lastTransitionTime: '2022-03-29T01:07:40Z'
      message: Deployment config has minimum availability.
  replicas: 1
  readyReplicas: 1
