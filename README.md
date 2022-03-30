# apigee-emulator-targetservers
Minimal PoC for using emulator with targetservers question

Trying to use TargetServers to decouple concrete endpoint URLs from TargetEndpoint configurations, as specified here: https://cloud.google.com/apigee/docs/api-platform/deploy/load-balancing-across-backend-servers

When developing locally using VSCode plugin. Apigee runtime version: 1.6.1, I'm getting "plpain HTTP request was sent to HTTPS port" when trying to hit HTTPS backends.

Restarting the emulator didn't help.


[apiproxies/sample-v1/apiproxy/targets/default.xml](./src/main/apigee/apiproxies/sample-v1/apiproxy/targets/default.xml):

```xml
<TargetEndpoint name="default">
    <HTTPTargetConnection>
        <LoadBalancer>
            <Server name="sample-backend"/>
        </LoadBalancer>
    </HTTPTargetConnection>
</TargetEndpoint>
```


[environments/local/targetservers.json](./src/main/apigee/environments/local/targetservers.json):

```json
[{
  "name": "sample-backend",
  "host": "httpbin.org",
  "port": 443,
  "sSLInfo": {
    "enabled": "true"
  }
}]
```

When issuing this command:

```bash
curl -v http://127.0.0.1:8998/sample/v1/get
```

I'm getting the following response:

```
*   Trying 127.0.0.1:8998...
* Connected to 127.0.0.1 (127.0.0.1) port 8998 (#0)
> GET /sample/v1/get HTTP/1.1
> Host: 127.0.0.1:8998
> User-Agent: curl/7.79.1
> Accept: */*
>
* Mark bundle as not supporting multiuse
< HTTP/1.1 400 Bad Request
< Server: awselb/2.0
< Date: Wed, 30 Mar 2022 09:54:30 GMT
< Content-Type: text/html
< Content-Length: 220
< X-Apigee-Message-ID: 9cb22ade-5b57-46fd-8379-b875591035fc8
< x-request-id: 9cb22ade-5b57-46fd-8379-b875591035fc8
< X-Apigee-organization: hybrid
< X-Apigee-proxy: /organizations/hybrid/environments/local/apiproxies/sample-v1/revisions/0
< X-Apigee-proxy-basepath: /sample/v1
< X-Apigee-fault-flag: false
< X-Apigee-fault-source: target
< X-Apigee-fault-code: messaging.adaptors.http.flow.ErrorResponseCode
< X-Apigee-fault-policy: null/null
< X-Apigee-fault-revision: /organizations/hybrid/environments/local/apiproxies/sample-v1/revisions/0
< X-Apigee-target-latency: 336
<
<html>
<head><title>400 The plain HTTP request was sent to HTTPS port</title></head>
<body>
<center><h1>400 Bad Request</h1></center>
<center>The plain HTTP request was sent to HTTPS port</center>
</body>
</html>
* Connection #0 to host 127.0.0.1 left intact
```

Post on Apigee forum: https://www.googlecloudcommunity.com/gc/Apigee/Getting-quot-plain-HTTP-request-was-sent-to-HTTPS-port-quot-when/m-p/408443
