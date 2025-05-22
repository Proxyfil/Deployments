# NTFY pour l'alerting

## Comment lancer le projet ?

Pour lancer l'application NTFY sur un cluster il suffit d'exécuter la commande suivante :
```bash
kubectl kustomize ./overlays/development/ | kubectl apply -f - --kubeconfig="chemin_vers_config_kube"
```

Ceci lancera l'application dans le namespace "ntfy"

## Comment créer un utilisateur ?

Pour créer un utilisateur nous devons nous rendre dans le shell du pod et exécuter la commande :
```bash
ntfy user add [user]
```

Il faudra ensuite rentrer le mot de passe de l'utilisateur

Pour le reste de commande vous pouvez aller voir la (documentation de NTFY)[https://docs.ntfy.sh/config/]

## Comment arrêter le projet ?

Pour arrêter l'application NTFY sur un cluster il suffit d'exécuter la commande suivante :
```bash
kubectl kustomize ./overlays/development/ | kubectl delete -f - --kubeconfig="chemin_vers_config_kube"
```