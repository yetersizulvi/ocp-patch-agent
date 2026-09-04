# macOS Build and Public Quay Push

Image macOS cihazında build edilir ve public Quay repository'ye gönderilir.
Kurum cluster'ı, endpoint'leri, registry prefix'i, model ve token değerleri image
içine yazılmaz. Bunlar OpenShift deployment aşamasında ConfigMap ve Secret ile
verilir.

## 1. Değişkenler

```bash
export QUAY_USER="REPLACE_WITH_QUAY_LOGIN_USER"
export QUAY_IMAGE="quay.io/REPLACE_WITH_QUAY_NAMESPACE/openshift-ai-assistant:0.5.1"
```

## 2. Quay login

```bash
podman login quay.io --username "${QUAY_USER}"
```

## 3. Lokal build

Bu komut kaynak dizininin içinde çalıştırılır:

```bash
podman build \
  --file Dockerfile \
  --tag "${QUAY_IMAGE}" \
  .
```

## 4. Image içeriği doğrulaması

```bash
podman run --rm "${QUAY_IMAGE}" \
  python -c 'from app import __version__; from app.master import DASHBOARD_HTML; assert "OpenShift AI Assistant" in DASHBOARD_HTML; print(__version__)'
```

Beklenen sürüm:

```text
0.5.1
```

## 5. Public Quay push

```bash
podman push "${QUAY_IMAGE}"
```

Repository public yapıldıktan sonra anonim erişimi doğrulayın:

```bash
podman logout quay.io
podman pull "${QUAY_IMAGE}"
```

## 6. OpenShift deployment

Deployment işlemleri bastion üzerinden yapılır. `deploy/` altındaki manifestlerde
aşağıdaki değerler kuruma göre deployment sırasında doldurulur:

```text
PATCH_AGENT_CLUSTER_ID
PATCH_AGENT_ENVIRONMENT
PATCH_AGENT_NAMESPACE_PATTERN
PATCH_AGENT_MASTER_URL
PATCH_AGENT_REGISTRY_PREFIX
LLM_URL
LLM_MODEL
AGENT_TOKEN
LLM_TOKEN
internal CA bundle
Quay image adı
```

Bu değerlerin hiçbiri public image build context'ine eklenmemelidir.
