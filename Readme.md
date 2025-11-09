# Redis HA Helm Chart

Ce Helm Chart déploie un **cluster Redis haute disponibilité (HA)** sur Kubernetes.  
Il met en place une architecture robuste composée d’un **Redis maître**, de **réplicas**, d’un système de supervision avec **Sentinel**, et d’un **exporter Prometheus** pour la supervision des métriques.

---

## Composants déployés

| Élément | Type Kubernetes | Rôle |
|----------|----------------|------|
| **Redis StatefulSet** | `StatefulSet` | Déploie les instances Redis principales et leurs réplicas. Les volumes persistants (PVC) permettent de conserver les données RDB/AOF entre redémarrages. |
| **Sentinel Deployment** | `Deployment` | Supervise les instances Redis, détecte les pannes et promeut un replica en tant que nouveau master en cas de failover. |
| **Sentinel Service** | `Service` | Permet aux clients et aux autres Sentinels de communiquer avec les pods Sentinel. |
| **Redis Exporter Deployment** | `Deployment` | Déploie le conteneur `oliver006/redis_exporter`, qui expose les métriques Redis pour Prometheus. |
| **Redis Exporter Service** | `Service` | Expose les métriques du Redis Exporter sur le port `9121`, généralement scrappé par Prometheus. |
| **ConfigMap Sentinel** | `ConfigMap` | Contient le fichier `sentinel.conf`, précisant la configuration Sentinel : le nom du master, le quorum, le timeout, et les credentials Redis. |
| **Secret redis-auth** | `Secret` | Stocke les identifiants Redis (mot de passe, etc.) utilisés à la fois par Redis, Sentinel et l’exporter. |
| **PVC Redis** | `PersistentVolumeClaim` | Fournit un stockage persistant pour les données Redis (`dump.rdb`, `appendonly.aof`). |

---

## Variables principales (`values.yaml`)

| Variable | Description | Exemple |
|-----------|--------------|----------|
| `auth.enabled` | Active l’authentification Redis | `true` |
| `auth.password` | Mot de passe Redis partagé entre Redis et Sentinel | `"redispassword"` |
| `persistence.enabled` | Active la persistance des données sur disque | `true` |
| `persistence.size` | Taille du volume persistant Redis | `"5Gi"` |
| `sentinel.replicas` | Nombre de pods Sentinel déployés | `3` |
| `exporter.enabled` | Active le déploiement du Redis Exporter | `true` |
| `exporter.namespace` | Namespace cible pour l’exporter (si différent du reste) | `"monitoring"` |
| `resources.requests.memory` | Mémoire demandée par Redis | `"4Gi"` |

---

## 🧩 Installation

```bash
helm install redis-ha ./redis-cluster-ha -n redis-prod
