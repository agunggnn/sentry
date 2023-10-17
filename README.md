# sentry on k3s helm install

  add sentry helm repo
  ```bash
  helm repo add sentry https://sentry-kubernetes.github.io/charts
  ```
  create file value.yml
  ```bash
  user:
    create: true
    email: agung.gunawan@shopee.com
    password: admin123!@#

  ingress:
    enabled: true
    # If you are using traefik ingress controller, switch this to 'traefik'
    regexPathStyle: traefik
    annotations:
    # If you are using nginx ingress controller, please use at least those 2 annotations
      spec.ingressClassName: traefik
      ingress.kubernetes.io/ssl-redirect: "false" 
    hostname: k3s.games.shopee.io

  sentry:
    singleOrganization: false
    worker:
      replicas: 2
  mail:
    backend: smtp
    useTls: false
    username: "apikey"
    password: "XXXXX"
    port: 25
    host: "smtp.sendgrid.net"
    from: "sentry@example.com"

  service:
    name: sentry
    type: ClusterIP
    externalPort: 9000
    annotations: {}
  slack: 
    clientId: "client-it"
    clientSecret: "cleint-secret"
    signingSecret: "signing-secret"
  # Reference -> https://develop.sentry.dev/integrations/slack/

  postgresql:
    enabled: true
  ## This value is only used when postgresql.enabled is set to false
  ##
  externalPostgresql:
    host: database-host
    port: 5432
    username: postgres
    password: ""
    database: sentry
  ```
  deploy sentry
  ```bash
  kubectl create namesapce sentry
  helm install --namespace sentry sentry -f ./value.yml sentry/sentry

  ## optional if error on db init
  # check db password encoded then decode
  oc get secret sentry-sentry-postgresql -oyaml
  
  echo "<encoded-string>" | base64 --decode
  
  kubectl get pods|grep postgresql
  kubectl exec -it sentry-sentry-postgresql-0 -- bash
  
  # reset password postgresql
  sed -ibak 's/^\([^#]*\)md5/\1trust/g' /opt/bitnami/postgresql/conf/pg_hba.conf
  pg_ctl reload
  
  psql -U postgres
  postgres=# alter user postgres with password 'NEW_PASSWORD';
  postgresl=# \q
  
  sed -i 's/^\([^#]*\)trust/\1md5/g' /opt/bitnami/postgresql/conf/pg_hba.conf
  pg_ctl reload

  helm upgrade -i --namespace sentry sentry -f value.yaml sentry/sentry
  ```
