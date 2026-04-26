# lesson-07-Yevhen-Marholin

## Deployment з 10 репліками

Було створено `Deployment` з 10 репліками.

```bash
kubectl get pods
kubectl get deployment
kubectl get rs
kubectl rollout status deployment/course-app
NAME                          READY   STATUS              RESTARTS   AGE
course-app-7487f857c4-228tm   0/1     ContainerCreating   0          8s
course-app-7487f857c4-2hmgx   1/1     Running             0          8s
course-app-7487f857c4-4cxjb   0/1     ContainerCreating   0          8s
course-app-7487f857c4-6x72j   1/1     Running             0          8s
course-app-7487f857c4-gj9vn   0/1     ContainerCreating   0          8s
course-app-7487f857c4-lss7z   1/1     Running             0          8s
course-app-7487f857c4-mqzl9   0/1     ContainerCreating   0          8s
course-app-7487f857c4-pwhk9   1/1     Running             0          8s
course-app-7487f857c4-rldgh   1/1     Running             0          8s
course-app-7487f857c4-rxdll   0/1     ContainerCreating   0          8s

NAME         READY   UP-TO-DATE   AVAILABLE   AGE
course-app   5/10    10           5           9s

NAME                    DESIRED   CURRENT   READY   AGE
course-app-7487f857c4   10        10        5       9s
deployment "course-app" successfully rolled out

```
## Оновлення ConfigMap і Pods

Було застосовано `ConfigMap`:

```bash
kubectl apply -f configmap.yaml
```

Після цього було виконано restart Deployment, щоб Pods отримали оновлені значення з ConfigMap:

```bash
kubectl rollout restart deployment/course-app
```

Перевірка rollout:

```bash
kubectl rollout status deployment/course-app
```

Результат:

```bash
deployment "course-app" successfully rolled out
```

Після оновлення були створені нові Pods:

```bash
course-app-5b99998469-4s9th   1/1   Running
course-app-5b99998469-7vwkg   1/1   Running
course-app-5b99998469-8kwd5   1/1   Running
course-app-5b99998469-ghvmg   1/1   Running
course-app-5b99998469-hsq6s   1/1   Running
course-app-5b99998469-jwjgf   1/1   Running
course-app-5b99998469-mlt7v   1/1   Running
course-app-5b99998469-ms2fg   1/1   Running
course-app-5b99998469-rrd8r   1/1   Running
course-app-5b99998469-xxq2p   1/1   Running
```

Старі Pods були поступово завершені:

```bash
course-app-7487f857c4-gj9vn   1/1   Terminating
course-app-7487f857c4-mqzl9   1/1   Terminating
```

Це показує, що Pods оновлюються поступово через механізм Deployment rollout.

## Оновлення образу контейнера

Було зібрано новий Docker image:

```bash
docker build -t course-app .
```

Image було затеговано новою версією:

```bash
docker tag course-app djmen12/course-app:lesson-07-v2
```

Image було завантажено у Docker Hub:

```bash
docker push djmen12/course-app:lesson-07-v2
```

Deployment було оновлено на новий image:

```bash
kubectl set image deployment/course-app course-app=djmen12/course-app:lesson-07-v2
```

Перевірка rollout:

```bash
kubectl rollout status deployment/course-app
```

Результат:

```bash
Waiting for deployment "course-app" rollout to finish: 2 old replicas are pending termination...
Waiting for deployment "course-app" rollout to finish: 1 old replicas are pending termination...
Waiting for deployment "course-app" rollout to finish: 9 of 10 updated replicas are available...
deployment "course-app" successfully rolled out
```

Після оновлення було створено новий ReplicaSet:

```bash
kubectl get rs
```

```bash
NAME                    DESIRED   CURRENT   READY   AGE
course-app-5b99998469   0         0         0       2m17s
course-app-7487f857c4   0         0         0       5m5s
course-app-d7d69d495    10        10        10      19s
```

Нові Pods працюють успішно:

```bash
kubectl get pods
```

```bash
course-app-d7d69d495-9vbhw   1/1   Running
course-app-d7d69d495-czpdp   1/1   Running
course-app-d7d69d495-hjp92   1/1   Running
course-app-d7d69d495-k9jhv   1/1   Running
course-app-d7d69d495-m2mzh   1/1   Running
course-app-d7d69d495-p68rb   1/1   Running
course-app-d7d69d495-sjv6x   1/1   Running
course-app-d7d69d495-vn59s   1/1   Running
course-app-d7d69d495-wgc67   1/1   Running
course-app-d7d69d495-wm9wm   1/1   Running
```

## RollingUpdate: maxUnavailable 1, maxSurge 1

У `deployment.yaml` було встановлено:

```yaml
strategy:
  type: RollingUpdate
  rollingUpdate:
    maxUnavailable: 1
    maxSurge: 1
```

Після цього Deployment було оновлено:

```bash
kubectl apply -f deployment.yaml
kubectl rollout restart deployment/course-app
kubectl rollout status deployment/course-app
```

Під час rollout Kubernetes поступово оновлював Pods:

```bash
Waiting for deployment "course-app" rollout to finish: 2 out of 10 new replicas have been updated...
Waiting for deployment "course-app" rollout to finish: 5 out of 10 new replicas have been updated...
Waiting for deployment "course-app" rollout to finish: 9 out of 10 new replicas have been updated...
Waiting for deployment "course-app" rollout to finish: 1 old replicas are pending termination...
deployment "course-app" successfully rolled out
```

Новий ReplicaSet став активним:

```bash
NAME                    DESIRED   CURRENT   READY   AGE
course-app-8c77d665f    10        10        10      13s
course-app-d7d69d495    0         0         0       2m36s
```

Висновок: з `maxUnavailable: 1` і `maxSurge: 1` оновлення проходить обережно. Kubernetes створює невелику кількість нових Pods і поступово завершує старі, зберігаючи доступність застосунку.

## RollingUpdate: maxUnavailable 0, maxSurge 3

У `deployment.yaml` було встановлено:

```yaml
strategy:
  type: RollingUpdate
  rollingUpdate:
    maxUnavailable: 0
    maxSurge: 3
```

Після цього Deployment було оновлено:

```bash
kubectl apply -f deployment.yaml
kubectl rollout restart deployment/course-app
kubectl rollout status deployment/course-app
```

Під час rollout Kubernetes створював нові Pods, не зменшуючи кількість доступних старих Pods:

```bash
Waiting for deployment "course-app" rollout to finish: 3 out of 10 new replicas have been updated...
Waiting for deployment "course-app" rollout to finish: 6 out of 10 new replicas have been updated...
Waiting for deployment "course-app" rollout to finish: 9 out of 10 new replicas have been updated...
Waiting for deployment "course-app" rollout to finish: 2 old replicas are pending termination...
deployment "course-app" successfully rolled out
```

Новий ReplicaSet:

```bash
NAME                    DESIRED   CURRENT   READY   AGE
course-app-75d85ff9bb   10        10        10      12s
course-app-8c77d665f    0         0         0       3m19s
```

Висновок: з `maxUnavailable: 0` і `maxSurge: 3` Kubernetes не вимикає старі Pods, поки не створить нові. Це безпечніше для доступності, але тимчасово використовує більше ресурсів.

