# K8Sgitops-monitoring

Dépôt GitOps pour déployer une stack de monitoring sur K3s avec Argo CD et Helm.

## Structure

- `apps/monitoring.yaml` : ressource Argo CD Application
- `monitoring/Chart.yaml` : chart Helm parent
- `monitoring/values.yaml` : configuration détaillée de la stack

## Composants déployés

Le chart `monitoring` dépend de `kube-prometheus-stack` (Prometheus Community).

Composants activés :

- Prometheus
- Grafana
- Alertmanager
- Node Exporter
- Kube State Metrics

Composants désactivés pour compatibilité K3s :

- kubeControllerManager
- kubeScheduler
- kubeProxy
- kubeEtcd

## Exposition des services

Définie dans `monitoring/values.yaml` :

- Prometheus : NodePort `30900`
- Grafana : NodePort `30300`
- Alertmanager : NodePort `30903`

## Déploiement GitOps (Argo CD)

Le manifeste `apps/monitoring.yaml` :

- source : ce dépôt Git
- chemin chart : `monitoring`
- namespace cible : `monitoring`
- sync automatique activée (prune + selfHeal)
- options :
  - `CreateNamespace=true`
  - `ServerSideApply=true` (important pour certains objets Prometheus sur K3s)

Application :

```bash
kubectl apply -f apps/monitoring.yaml
```

## Déploiement manuel avec Helm (optionnel)

1. Ajouter le repo de dépendance :

```bash
helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
helm repo update
```

2. Déployer :

```bash
kubectl create namespace monitoring
helm dependency update monitoring
helm upgrade --install monitoring monitoring -n monitoring
```

3. Vérifier :

```bash
kubectl get pods,svc -n monitoring
```

## Notes K3s

- Certaines métriques Kubelet peuvent nécessiter des ajustements selon la distribution K3s.
- La configuration actuelle réduit les erreurs TLS et les conflits de composants non présents sur K3s.
- Le mot de passe admin Grafana défini dans values est un exemple et doit être changé en environnement réel.
