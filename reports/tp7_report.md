# TP7 - Rapport court

## 1) Modèle utilisé
- Source des poids (TP4/TP5/TP6) : TP6 (best.pt)
- Nom du run MLflow (si applicable) : N/A (fichier local)
- Fichier utilisé : `models/weights/best.pt`

## 2) Packaging
- Option utilisée : **Docker**
- Commande(s) exécutée(s) :
  ```bash
  docker run --rm -v ${PWD}:/workspace -w /workspace salahgo/torchserve:latest-cpu torch-model-archiver --model-name yolo --version 1.0 ...
  ```
- Résultat : yolo.mar généré (**Oui**)

## 3) Déploiement
- `docker compose up -d --build`
- Services démarrés : **torchserve**, **api-gateway**

## 4) Tests
- Test TorchServe : **Succès** (Objets détectés : person, truck, etc.)
- Test Gateway : **Succès** (Même résultat JSON)
- Exemple de sortie JSON (court extrait) :
  ```json
  [
    {"boxes": [
      {"name": "person", "conf": 0.85, "xyxy": [73.8, 231.3, 103.4, 320.0]},
      {"name": "truck", "conf": 0.91, "xyxy": [255.1, 201.9, 305.8, 246.7]}
    ]}
  ]
  ```

## 5) Redéploiement
- Nouvelle version de poids : Simulation (même fichier `best.pt`, version 2.0)
- Étapes effectuées :
  1. Backup `yolo.mar.v1`
  2. Régénération `yolo.mar` (v2.0)
  3. `docker compose restart torchserve`
- Résultat : Service redémarré avec la nouvelle version.

## 6) Discussion
- **Pourquoi une gateway rend l’architecture plus agnostique ?**
  Elle masque la spécificité de l'API TorchServe (`/predictions/{model_name}`). Le client consomme un endpoint générique `/predict`. Si on change TorchServe pour TensorFlow Serving ou ONNX Runtime, seul l'API Gateway change, pas le client.
- **Limites rencontrées :**
  Latence CPU (image officielle CPU). Inférence synchrone bloquante.
- **Améliorations possibles :**
  Utilisation de GPU. Mode asynchrone (Celery/Redis ou Kafka). Passage à KServe pour le scaling automatique sur Kubernetes.
