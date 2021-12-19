# keycloak-with-opentelemetry

Single click provisioning
[![Gitpod ready-to-test](https://img.shields.io/badge/Gitpod-ready--to--test-blue?logo=gitpod)](https://gitpod.io/#https://github.com/jangaraj/keycloak-with-opentelemetry/)
- Keycloak login: `admin/admin`
- Keycloak metric SPI metrics: `/auth/realms/master/metrics` (disabled for a moment)
- Keycloak metrics: `:9990/metrics`


## Performance testing

Tests were executed against single dockerized Keycloak 15.0.2 (PostgreSQL DB, no LDAP, default client config - not optimized, without TLS offloading).
Keycloak deployed on 4vCPU (with AES, SHA HW support), 8GB RAM. Cross datacenter testing. K6 1min testing - 1st test run was ingored, it was used
to load data into caches.
Opentelemetry agent 1.6.2 enabled/disabled via env variable `JAVA_OPTS_APPEND: "-javaagent:/tmp/opentelemetry-javaagent-all.jar"`

### Token endpoint without opentelemetry agent

```
# k6 test output:
     checks.........................: 100.00% ? 9936      ? 0
     data_received..................: 19 MB   295 kB/s
     data_sent......................: 1.2 MB  20 kB/s
     http_req_blocked...............: avg=26.69ms  min=260ns    med=610ns    max=2.18s   p(90)=870ns    p(95)=1.13탎
     http_req_connecting............: avg=1.66ms   min=0s       med=0s       max=62.06ms p(90)=0s       p(95)=0s
     http_req_duration..............: avg=2.43s    min=96.11ms  med=2.46s    max=2.84s   p(90)=2.6s     p(95)=2.64s
       { expected_response:true }...: avg=2.43s    min=96.11ms  med=2.46s    max=2.84s   p(90)=2.6s     p(95)=2.64s
     http_req_failed................: 0.00%   ? 0         ? 4969
     http_req_receiving.............: avg=98.6탎   min=28.23탎  med=86.75탎  max=2.68ms  p(90)=120.66탎 p(95)=181.7탎
     http_req_sending...............: avg=124.16탎 min=50.5탎   med=102.68탎 max=4.83ms  p(90)=185.76탎 p(95)=237.29탎
     http_req_tls_handshaking.......: avg=25.01ms  min=0s       med=0s       max=2.15s   p(90)=0s       p(95)=0s
     http_req_waiting...............: avg=2.43s    min=95.67ms  med=2.46s    max=2.84s   p(90)=2.6s     p(95)=2.64s
     http_reqs......................: 4969    79.466452/s
     iteration_duration.............: avg=2.46s    min=232.09ms med=2.47s    max=4.59s   p(90)=2.61s    p(95)=2.66s
     iterations.....................: 4968    79.450459/s
     vus............................: 58      min=58      max=200
     vus_max........................: 200     min=200     max=200

# docker stats output
CONTAINER ID   NAME                  CPU %     MEM USAGE / LIMIT     MEM %     NET I/O       BLOCK I/O        PIDS
93a764395bf6   keycloak_keycloak_1   380.30%   739.2MiB / 7.775GiB   9.28%     10.4MB / 32.7MB   121MB / 1.64MB   180
```
### Token endpoint with opentelemetry agent

```
# k6 test output:
     checks.........................: 100.00% ? 9980      ? 0
     data_received..................: 19 MB   297 kB/s
     data_sent......................: 1.2 MB  20 kB/s
     http_req_blocked...............: avg=29.7ms   min=240ns    med=580ns    max=2.38s    p(90)=840ns    p(95)=1.06탎
     http_req_connecting............: avg=2.99ms   min=0s       med=0s       max=122.93ms p(90)=0s       p(95)=0s
     http_req_duration..............: avg=2.41s    min=79.21ms  med=2.44s    max=2.94s    p(90)=2.61s    p(95)=2.67s
       { expected_response:true }...: avg=2.41s    min=79.21ms  med=2.44s    max=2.94s    p(90)=2.61s    p(95)=2.67s
     http_req_failed................: 0.00%   ? 0         ? 4991
     http_req_receiving.............: avg=95.07탎  min=28.45탎  med=86.56탎  max=1.5ms    p(90)=117.89탎 p(95)=172.96탎
     http_req_sending...............: avg=119.28탎 min=50.94탎  med=102.42탎 max=2.3ms    p(90)=171.78탎 p(95)=217.97탎
     http_req_tls_handshaking.......: avg=26.65ms  min=0s       med=0s       max=2.29s    p(90)=0s       p(95)=0s
     http_req_waiting...............: avg=2.41s    min=78.81ms  med=2.44s    max=2.94s    p(90)=2.61s    p(95)=2.67s
     http_reqs......................: 4991    80.005383/s
     iteration_duration.............: avg=2.44s    min=201.78ms med=2.44s    max=5s       p(90)=2.62s    p(95)=2.7s
     iterations.....................: 4990    79.989353/s
     vus............................: 40      min=40      max=200
     vus_max........................: 200     min=200     max=200

# docker stats output
CONTAINER ID   NAME                  CPU %     MEM USAGE / LIMIT     MEM %     NET I/O           BLOCK I/O        PIDS
6a14f007517a   keycloak_keycloak_1   380.31%   913.3MiB / 7.775GiB   11.47%    13.7MB / 42.9MB   151MB / 1.73MB   188
```


### Static asset without opentelemetry agent

```
# k6 test output:
     checks.........................: 100.00% ? 174899      ? 0
     data_received..................: 1.3 GB  20 MB/s
     data_sent......................: 16 MB   238 kB/s
     http_req_blocked...............: avg=34.02ms  min=240ns   med=570ns    max=4.47s    p(90)=720ns    p(95)=840ns
     http_req_connecting............: avg=1.66ms   min=0s      med=0s       max=385.42ms p(90)=0s       p(95)=0s
     http_req_duration..............: avg=656.39ms min=27.84ms med=347.52ms max=19.79s   p(90)=1.24s    p(95)=2.53s
       { expected_response:true }...: avg=656.39ms min=27.84ms med=347.52ms max=19.79s   p(90)=1.24s    p(95)=2.53s
     http_req_failed................: 0.00%   ? 0           ? 174900
     http_req_receiving.............: avg=15.33ms  min=43.4탎  med=1.07ms   max=489.41ms p(90)=53.13ms  p(95)=70.27ms
     http_req_sending...............: avg=320.42탎 min=26.98탎 med=47.84탎  max=473.56ms p(90)=192.44탎 p(95)=263.86탎
     http_req_tls_handshaking.......: avg=32.34ms  min=0s      med=0s       max=4.34s    p(90)=0s       p(95)=0s
     http_req_waiting...............: avg=640.73ms min=0s      med=326.11ms max=19.78s   p(90)=1.22s    p(95)=2.49s
     http_reqs......................: 174900  2587.745399/s
     iteration_duration.............: avg=692.67ms min=27.95ms med=349.84ms max=19.79s   p(90)=1.39s    p(95)=3.78s
     iterations.....................: 174899  2587.730604/s
     vus............................: 94      min=94        max=2000
     vus_max........................: 2000    min=2000      max=2000

# docker stats output
ONTAINER ID   NAME                  CPU %     MEM USAGE / LIMIT     MEM %     NET I/O          BLOCK I/O        PIDS
eeb743dde570   keycloak_keycloak_1   243.42%   804.1MiB / 7.775GiB   10.10%    145MB / 3.09GB   217kB / 1.64MB   179
```
### Static asset with opentelemetry agent

```
# k6 test output:
     checks.........................: 100.00% ? 174812      ? 0
     data_received..................: 1.3 GB  20 MB/s
     data_sent......................: 16 MB   237 kB/s
     http_req_blocked...............: avg=29.72ms  min=240ns   med=570ns    max=3.92s    p(90)=730ns    p(95)=850ns
     http_req_connecting............: avg=2.31ms   min=0s      med=0s       max=630.43ms p(90)=0s       p(95)=0s
     http_req_duration..............: avg=657.42ms min=28.23ms med=359.44ms max=19.47s   p(90)=1.26s    p(95)=2.44s
       { expected_response:true }...: avg=657.42ms min=28.23ms med=359.44ms max=19.47s   p(90)=1.26s    p(95)=2.44s
     http_req_failed................: 0.00%   ? 0           ? 174813
     http_req_receiving.............: avg=15.74ms  min=45.26탎 med=1.22ms   max=417.32ms p(90)=53.82ms  p(95)=71.02ms
     http_req_sending...............: avg=328.06탎 min=26.86탎 med=47.74탎  max=582.28ms p(90)=194.19탎 p(95)=263.65탎
     http_req_tls_handshaking.......: avg=27.39ms  min=0s      med=0s       max=3.73s    p(90)=0s       p(95)=0s
     http_req_waiting...............: avg=641.35ms min=5.92ms  med=338.8ms  max=19.42s   p(90)=1.23s    p(95)=2.39s
     http_reqs......................: 174813  2577.263932/s
     iteration_duration.............: avg=687.48ms min=28.34ms med=359.98ms max=19.47s   p(90)=1.37s    p(95)=3.4s
     iterations.....................: 174812  2577.249189/s
     vus............................: 10      min=10        max=2000
     vus_max........................: 2000    min=2000      max=2000

# docker stats output
CONTAINER ID   NAME                  CPU %     MEM USAGE / LIMIT     MEM %     NET I/O         BLOCK I/O        PIDS
6a14f007517a   keycloak_keycloak_1   348.61%   1.147GiB / 7.775GiB   14.76%    369MB / 4.5GB   154MB / 1.73MB   187
```

Reference app (Caddy 2 with static response):
```
     checks.........................: 100.00% ? 276108      ? 0
     data_received..................: 28 MB   464 kB/s
     data_sent......................: 12 MB   191 kB/s
     http_req_blocked...............: avg=22.61ms  min=230ns   med=550ns    max=3.83s p(90)=690ns    p(95)=800ns
     http_req_connecting............: avg=8.33ms   min=0s      med=0s       max=2.5s  p(90)=0s       p(95)=0s
     http_req_duration..............: avg=623.88ms min=33.89ms med=529.02ms max=2.5s  p(90)=1.02s    p(95)=1.27s
       { expected_response:true }...: avg=623.88ms min=33.89ms med=529.02ms max=2.5s  p(90)=1.02s    p(95)=1.27s
     http_req_failed................: 0.00%   ? 0           ? 276108
     http_req_receiving.............: avg=26.17ms  min=19.71탎 med=23.09ms  max=1.41s p(90)=45.73ms  p(95)=61.2ms
     http_req_sending...............: avg=506.91탎 min=28.82탎 med=51.5탎   max=1.44s p(90)=122.46탎 p(95)=233.88탎
     http_req_tls_handshaking.......: avg=13.8ms   min=0s      med=0s       max=2.89s p(90)=0s       p(95)=0s
     http_req_waiting...............: avg=597.2ms  min=28.72ms med=506.04ms max=2.48s p(90)=984.26ms p(95)=1.22s
     http_reqs......................: 276108  4579.192526/s
     iteration_duration.............: avg=649.55ms min=36.07ms med=530.34ms max=4.25s p(90)=1.06s    p(95)=1.32s
     iterations.....................: 276108  4579.192526/s
     vus............................: 2907    min=2907      max=3000
     vus_max........................: 3000    min=3000      max=3000
```

### Performance conclusion

There is minimal performance degradation in the Keycloak from the user perspective when Open Telemetry agent is deployed.
