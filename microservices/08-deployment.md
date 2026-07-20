# Deployment

> От Docker-образа до продовой выкладки без даунтайма: как собрать образ, какими объектами Kubernetes управляет сервисом, и три стратегии выкатки нового кода с разным риском.

## Содержание
- [Docker](#docker)
- [Kubernetes — ключевые объекты](#kubernetes--ключевые-объекты)
- [Стратегии деплоя](#стратегии-деплоя)
- [Feature Flags](#feature-flags)
- [Подводные камни](#подводные-камни)
- [См. также](#см-также)

## Docker

Multi-stage build даёт минимальный итоговый образ — SDK нужен только на этапе сборки, в runtime-образе его нет.

```dockerfile
FROM mcr.microsoft.com/dotnet/sdk:8.0 AS build
WORKDIR /src
COPY ["OrderService.csproj", "."]
RUN dotnet restore
COPY .. .
RUN dotnet publish -c Release -o /app/publish

FROM mcr.microsoft.com/dotnet/aspnet:8.0 AS runtime
WORKDIR /app
RUN adduser --disabled-password --gecos "" appuser # непривилегированный пользователь
USER appuser
COPY --from=build /app/publish .
EXPOSE 8080
ENTRYPOINT ["dotnet", "OrderService.dll"]
```

## Kubernetes — ключевые объекты

**Pod** — минимальная единица, один или несколько контейнеров с общей сетью и хранилищем.

**Deployment** — управляет Pod'ами: rolling update, rollback, желаемое число реплик.

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: order-service
spec:
  replicas: 3
  selector:
    matchLabels:
      app: order-service
  template:
    metadata:
      labels:
        app: order-service
    spec:
      containers:
      - name: order-service
        image: myregistry/order-service:1.2.3
        ports:
        - containerPort: 8080
        env:
        - name: ConnectionStrings__Default
          valueFrom:
            secretKeyRef:
              name: order-service-secrets
              key: db-connection-string
        - name: ASPNETCORE_ENVIRONMENT
          valueFrom:
            configMapKeyRef:
              name: order-service-config
              key: environment
        resources:
          requests:
            memory: "128Mi"
            cpu: "100m"
          limits:
            memory: "512Mi"
            cpu: "500m"
        livenessProbe:
          httpGet: { path: /health/live, port: 8080 }
          initialDelaySeconds: 30
          periodSeconds: 10
        readinessProbe:
          httpGet: { path: /health/ready, port: 8080 }
          initialDelaySeconds: 10
          periodSeconds: 5
```

Пробы ссылаются на эндпоинты из [Health Checks](./07-observability.md#health-checks).

**Service** — стабильное DNS-имя + virtual IP для доступа к Pod'ам:

```yaml
apiVersion: v1
kind: Service
metadata:
  name: order-service
spec:
  selector:
    app: order-service
  ports:
  - port: 80
    targetPort: 8080
  type: ClusterIP # внутренний; LoadBalancer для внешнего доступа
```

**ConfigMap и Secret:**

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: order-service-config
data:
  environment: "Production"
  kafka-brokers: "kafka:9092"
---
apiVersion: v1
kind: Secret
metadata:
  name: order-service-secrets
type: Opaque
stringData:
  db-connection-string: "Host=postgres;Database=orders;Username=app;Password=secret"
```

## Стратегии деплоя

**Rolling Update** (дефолт в Kubernetes) — поды обновляются постепенно, старый Pod убирается только когда новый прошёл readiness check:

```yaml
spec:
  strategy:
    type: RollingUpdate
    rollingUpdate:
      maxSurge: 1       # максимум 1 лишний Pod во время update
      maxUnavailable: 0 # 0 недоступных Pod'ов — zero-downtime
```

**Blue-Green:**

```
Blue (v1.0): 3 replicas ← трафик
Green (v2.0): 3 replicas ← нет трафика, тестируем

Переключение: Service selector → green
Если что-то не так: Service selector → blue (мгновенный rollback)
```

Требует двойные ресурсы на время деплоя. Чистый переключатель без постепенного rollout.

**Canary:**

```
v1.0: 90% трафика
v2.0: 10% трафика (canary)

Мониторинг: error rate, latency у canary vs stable
Если OK → постепенно увеличиваем до 100%
Если плохо → откатываем canary
```

В Kubernetes реализуется через `weight` в Ingress или через Service Mesh (Istio `VirtualService` — см. [Service Discovery и API Gateway](./06-gateway-discovery.md#service-mesh)).

## Feature Flags

Деплой кода отделён от включения функциональности — новую логику можно выкатить «выключенной» и включить позже без нового деплоя.

```csharp
builder.Services.AddFeatureManagement()
    .AddFeatureFilter<PercentageFilter>()  // % пользователей
    .AddFeatureFilter<TargetingFilter>();  // конкретные пользователи/группы
```

```json
{
  "FeatureManagement": {
    "NewCheckout": true,
    "ExperimentalPricing": {
      "EnabledFor": [
        { "Name": "Percentage", "Parameters": { "Value": 10 } }
      ]
    }
  }
}
```

```csharp
public class CheckoutService
{
    private readonly IFeatureManager _features;

    public async Task<CheckoutResult> CheckoutAsync(CartId cartId, CancellationToken ct)
    {
        if (await _features.IsEnabledAsync("NewCheckout"))
            return await NewCheckoutFlowAsync(cartId, ct);

        return await LegacyCheckoutAsync(cartId, ct);
    }
}
```

## Подводные камни

- `maxUnavailable: 0` без соответствующего `readinessProbe` — Kubernetes не знает, когда новый Pod реально готов принимать трафик, и может отправлять запросы в под, который ещё не поднялся.
- Blue-Green без синхронизации схемы БД — если Green использует новую миграцию, а Blue ещё работает со старой схемой, откат на Blue после переключения может сломаться из-за несовместимости данных.
- Canary без реального мониторинга метрик (error rate/latency) во время раската — «канарейка» превращается в обычный поэтапный rollout без сигнала для остановки при деградации.
- Feature Flags, которые никогда не удаляют после полного релиза — код обрастает мёртвыми ветками `if (flag) {...} else {...}`, где `else` никогда не выполняется.

## См. также

- [Observability](./07-observability.md) — health checks, на которых строятся Kubernetes probes
- [Service Discovery и API Gateway](./06-gateway-discovery.md) — Canary через Service Mesh
- [Антипаттерны](./09-antipatterns.md) — Big Bang Rewrite как противоположность постепенным стратегиям деплоя
