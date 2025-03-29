# hull-wrapper
[Helm Uniform Layer Library](https://github.com/vidispine/hull) wrapper

## Job example

```yaml
hull:
  config:
    general:
      nameOverride: duckdns-updater
      rbac: false
      noObjectNamePrefixes: true
  objects:
    serviceaccount:
      default:
        enabled: false
    job:
      duckdns-updater:
        enabled: true
        pod:
          containers:
            main:
              image:
                repository: curlimages/curl
                tag: latest
              command:
                - /bin/sh
                - -c
              args:
                - |
                  echo "Updating DuckDNS record for $DUCKDNS_DOMAIN..."
                  response=$(curl -sSL "https://www.duckdns.org/update?domains=$DUCKDNS_DOMAIN&token=$DUCKDNS_TOKEN")
                  if [ "$response" = "OK" ]; then
                      echo "Successfully updated DuckDNS record"
                      exit 0
                  else
                      echo "Failed to update DuckDNS record. response: $response"
                      exit 1
                  fi
              env:
                DUCKDNS_DOMAIN:
                  value: ${var.duckdns_domain}
                DUCKDNS_TOKEN:
                  valueFrom:
                    secretKeyRef:
                      name: duckdns-token
                      key: token
          restartPolicy: Never
```

## App example

```yaml
hull:
  config:
    general:
      nameOverride: pdf-generator
      rbac: false
      noObjectNamePrefixes: true
  objects:
    serviceaccount:
      default:
        enabled: false
    deployment:
      pdf-generator:
        replicas: 4
        annotations:
          linkerd.io/inject: enabled
          config.linkerd.io/proxy-log-level: debug
        pod:
          containers:
            main:
              resources:
                requests:
                  memory: 256Mi
                  cpu: 250m
                limits:
                  memory: 2048Mi
                  cpu: 2000m
              image:
                repository: ghcr.io/avdeev99/puppeteer-pdf-generator
                tag: 93420ed874e6937871ce6a40449a960aa8738e86
              env:
                CHROMIUM_PATH:
                  value: /usr/bin/chromium
                PUPPETEER_MAX_CONCURRENT_PAGES:
                  value: 15
                ASPNETCORE_URLS:
                  value: http://0.0.0.0:3000
              ports:
                http:
                  containerPort: 3000
    service:
      pdf-generator:
        type: ClusterIP
        ports:
          http:
            port: 3000
            targetPort: 3000
```
