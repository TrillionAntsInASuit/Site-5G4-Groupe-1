+++
title = "Atelier pratique : Voice-to-Text"
weight = 3
+++

---

## 📋 Objectifs du laboratoire

À la fin de ce laboratoire, vous serez capable de :
- Comprendre les bases de la reconnaissance vocale automatique (ASR)
- Installer et configurer un environnement Docker pour l'ASR
- Utiliser Whisper d'OpenAI pour transcrire de l'audio en texte
- Traiter différents formats audio et langues
- Créer un script Python pour automatiser la transcription

**Durée estimée**: 2h30 à 3h00

---

##  Prérequis

- Docker installé sur votre machine
- Fichiers audio de test (ou vous pouvez enregistrer votre voix)
- 4 GB d'espace disque disponible
- Connexion Internet pour télécharger les modèles
- Dépôt git: https://github.com/Marwan-Maalouf/atelier-ASR.git

---

##  Partie 1: Installation et configuration

**Durée**: 30-45 minutes

### Étape 1.1: Préparer l'environnement

Créez un dossier pour le laboratoire dans le repertoire:

```bash
mkdir lab-asr
cd lab-asr
mkdir audio_samples
mkdir transcriptions
```

### Étape 1.2: Créer le Dockerfile

Créez un fichier nommé `Dockerfile` avec le contenu suivant:

```dockerfile
# Image de base Ubuntu
FROM ubuntu:22.04

# Éviter les prompts interactifs
ENV DEBIAN_FRONTEND=noninteractive

# Mise à jour et installation des dépendances
RUN apt-get update && apt-get install -y \
    python3 \
    python3-pip \
    ffmpeg \
    git \
    nano \
    wget \
    curl \
    && rm -rf /var/lib/apt/lists/*

# Installation de Whisper et ses dépendances
RUN pip3 install --no-cache-dir \
    openai-whisper \
    pydub \
    torch \
    numpy

# Créer les répertoires de travail
WORKDIR /workspace
RUN mkdir -p /workspace/audio_samples /workspace/transcriptions

# Point d'entrée
CMD ["/bin/bash"]
```

### Étape 1.3: Construire l'image Docker

```bash
docker build -t lab-asr:latest .
```

{{< notice info >}}
 Cette étape peut prendre 5-10 minutes selon votre connexion Internet.
{{< /notice >}}

### Étape 1.4: Lancer le conteneur

```bash
docker run -it --rm \
  -v $(pwd)/audio_samples:/workspace/audio_samples \
  -v $(pwd)/transcriptions:/workspace/transcriptions \
  --name asr-lab \
  lab-asr:latest
```

{{< notice success >}}
**Point de vérification**: Vous devriez maintenant être dans le conteneur avec un prompt qui ressemble à `root@xxxxx:/workspace#`
{{< /notice >}}

---

##  Partie 2: Préparer des échantillons audio

**Durée**: 15-20 minutes

### Étape 2.1: Télécharger des échantillons audio de test

Dans le conteneur, téléchargez quelques exemples:

```bash
# Exemple en anglais
wget -O /audio_samples/Enregistrement.m4a \
  "https://www2.cs.uic.edu/~i101/SoundFiles/BabyElephantWalk60.wav"

# Vous pouvez aussi copier vos propres fichiers audio
# depuis votre machine hôte vers le dossier audio_samples
```

### Étape 2.2: Créer vos propres échantillons audio

Sortez du conteneur (tapez `exit`) et créez des fichiers audio de test.

{{< notice tip >}}
 **Conseil**: Enregistrez 2-3 phrases claires en français avec:
- **Audacity** (Linux)
- **GNOME Sound Recorder**
- Votre téléphone, puis transférez le fichier

Sauvegardez en MP3 ou WAV dans le dossier `audio_samples/`
{{< /notice >}}

**Exemples de phrases à enregistrer**:
- "Bonjour, ceci est un test de reconnaissance vocale automatique."
- "La technologie ASR permet de convertir la parole en texte."
- "Docker facilite le déploiement d'applications complexes."

---

##  Partie 3: Premiers pas avec Whisper

**Durée**: 30-40 minutes

Retournez dans le conteneur:

```bash
docker start -i asr-lab
# ou si vous l'aviez arrêté:
docker run -it --rm \
  -v $(pwd)/audio_samples:/workspace/audio_samples \
  -v $(pwd)/transcriptions:/workspace/transcriptions \
  --name asr-lab \
  lab-asr:latest
```

### Étape 3.1: Test basique avec Whisper CLI

```bash
# Transcrire un fichier audio
whisper /workspace/audio_samples/sample_en.mp3 --model tiny --language English
```

** Observations à noter**:
- Temps de traitement
- Précision de la transcription
- Fichiers générés (txt, json, srt, etc.)

### Étape 3.2: Comprendre les modèles Whisper

Whisper propose plusieurs tailles de modèles:

| Modèle | Paramètres | Taille | Vitesse | Précision |
|--------|-----------|--------|---------|-----------|
| tiny   | 39 M      | ~75 MB | Très rapide | Basique |
| base   | 74 M      | ~142 MB | Rapide | Bonne |
| small  | 244 M     | ~466 MB | Moyen | Très bonne |
| medium | 769 M     | ~1.5 GB | Lent | Excellente |
| large  | 1550 M    | ~3 GB | Très lent | Maximale |

{{< notice info >}}
💡 Pour ce laboratoire, nous utiliserons principalement **tiny** et **base** pour la rapidité.
{{< /notice >}}

---

## 💻Partie 4: Scripts Python pour la transcription

**Durée**: 45-60 minutes

### Exercice 1: Script de transcription simple

Créez un fichier `transcribe_simple.py`:

```python
#!/usr/bin/env python3
import whisper
import sys
import os

def transcribe_audio(audio_path, model_name="base"):
    """
    Transcrit un fichier audio en texte
    
    Args:
        audio_path: Chemin vers le fichier audio
        model_name: Nom du modèle Whisper à utiliser
    
    Returns:
        Texte transcrit
    """
    print(f" Chargement du modèle '{model_name}'...")
    model = whisper.load_model(model_name)
    
    print(f"🎤 Transcription de: {audio_path}")
    result = model.transcribe(audio_path)
    
    return result

if __name__ == "__main__":
    if len(sys.argv) < 2:
        print("Usage: python3 transcribe_simple.py <fichier_audio>")
        sys.exit(1)
    
    audio_file = sys.argv[1]
    
    if not os.path.exists(audio_file):
        print(f" Erreur: Le fichier '{audio_file}' n'existe pas")
        sys.exit(1)
    
    # Transcription
    result = transcribe_audio(audio_file)
    
    # Affichage des résultats
    print("\n" + "="*50)
    print(" TRANSCRIPTION:")
    print("="*50)
    print(result["text"])
    print("="*50)
    print(f" Langue détectée: {result['language']}")
```

**Testez le script**:

```bash
python3 transcribe_simple.py /workspace/audio_samples/sample_en.mp3
```

### Exercice 2: Script avancé avec sauvegarde et timestamps

Créez `transcribe_advanced.py`:

```python
#!/usr/bin/env python3
import whisper
import json
import os
from datetime import datetime

def transcribe_with_timestamps(audio_path, model_name="base", language=None):
    """
    Transcrit un fichier audio avec timestamps et sauvegarde les résultats
    """
    # Chargement du modèle
    print(f"🔄 Chargement du modèle '{model_name}'...")
    model = whisper.load_model(model_name)
    
    # Options de transcription
    options = {"fp16": False}
    if language:
        options["language"] = language
    
    # Transcription
    print(f"🎤 Transcription en cours...")
    result = model.transcribe(audio_path, **options)
    
    return result

def save_transcription(result, audio_path, output_dir="/workspace/transcriptions"):
    """
    Sauvegarde la transcription dans différents formats
    """
    # Créer le nom de base pour les fichiers de sortie
    base_name = os.path.splitext(os.path.basename(audio_path))[0]
    timestamp = datetime.now().strftime("%Y%m%d_%H%M%S")
    
    # 1. Sauvegarder le texte complet
    txt_path = os.path.join(output_dir, f"{base_name}_{timestamp}.txt")
    with open(txt_path, "w", encoding="utf-8") as f:
        f.write(result["text"])
    print(f" Texte sauvegardé: {txt_path}")
    
    # 2. Sauvegarder avec timestamps (format JSON)
    json_path = os.path.join(output_dir, f"{base_name}_{timestamp}.json")
    with open(json_path, "w", encoding="utf-8") as f:
        json.dump(result, f, ensure_ascii=False, indent=2)
    print(f" JSON sauvegardé: {json_path}")
    
    # 3. Sauvegarder au format SRT (sous-titres)
    srt_path = os.path.join(output_dir, f"{base_name}_{timestamp}.srt")
    write_srt(result["segments"], srt_path)
    print(f" SRT sauvegardé: {srt_path}")
    
    return txt_path, json_path, srt_path

def write_srt(segments, output_path):
    """
    Écrit un fichier SRT (sous-titres) à partir des segments
    """
    with open(output_path, "w", encoding="utf-8") as f:
        for i, segment in enumerate(segments, start=1):
            start_time = format_timestamp(segment["start"])
            end_time = format_timestamp(segment["end"])
            text = segment["text"].strip()
            
            f.write(f"{i}\n")
            f.write(f"{start_time} --> {end_time}\n")
            f.write(f"{text}\n\n")

def format_timestamp(seconds):
    """
    Formate les secondes en format SRT (HH:MM:SS,mmm)
    """
    hours = int(seconds // 3600)
    minutes = int((seconds % 3600) // 60)
    secs = int(seconds % 60)
    millis = int((seconds % 1) * 1000)
    return f"{hours:02d}:{minutes:02d}:{secs:02d},{millis:03d}"

def display_segments(segments):
    """
    Affiche les segments avec timestamps
    """
    print("\n" + "="*70)
    print(" TRANSCRIPTION AVEC TIMESTAMPS:")
    print("="*70)
    
    for segment in segments:
        start = segment["start"]
        end = segment["end"]
        text = segment["text"].strip()
        print(f"[{start:.2f}s → {end:.2f}s] {text}")
    
    print("="*70)

if __name__ == "__main__":
    import sys
    
    if len(sys.argv) < 2:
        print("Usage: python3 transcribe_advanced.py <fichier_audio> [modèle] [langue]")
        print("Exemple: python3 transcribe_advanced.py audio.mp3 base fr")
        sys.exit(1)
    
    audio_file = sys.argv[1]
    model_name = sys.argv[2] if len(sys.argv) > 2 else "base"
    language = sys.argv[3] if len(sys.argv) > 3 else None
    
    if not os.path.exists(audio_file):
        print(f" Erreur: Le fichier '{audio_file}' n'existe pas")
        sys.exit(1)
    
    # Transcription
    result = transcribe_with_timestamps(audio_file, model_name, language)
    
    # Affichage
    print(f"\n Langue détectée: {result['language']}")
    display_segments(result["segments"])
    
    # Sauvegarde
    print("\n Sauvegarde des résultats...")
    save_transcription(result, audio_file)
    
    print("\n Transcription terminée avec succès!")
```

**Testez le script avancé**:

```bash
python3 transcribe_advanced.py /workspace/audio_samples/sample_en.mp3 base en
```

---

##  Partie 5: Exercices pratiques

**Durée**: 30-45 minutes

### Exercice 3: Traitement par lot (Batch Processing)

**Objectif**: Créer un script qui transcrit tous les fichiers audio d'un dossier

**Instructions**:
1. Créez un fichier `batch_transcribe.py`
2. Le script doit:
   - Parcourir tous les fichiers .mp3, .wav, .m4a dans un dossier
   - Transcrire chaque fichier
   - Sauvegarder les résultats
   - Créer un rapport récapitulatif (nombre de fichiers, durée totale, etc.)

{{< notice tip >}}
** Indice**: Utilisez `os.listdir()` et une boucle `for` pour parcourir les fichiers
{{< /notice >}}

### Exercice 4: Détection de langue automatique

**Objectif**: Modifier le script pour détecter automatiquement la langue

**Instructions**:
1. Créez un fichier `detect_language.py`
2. Transcrivez le fichier SANS spécifier la langue
3. Affichez la langue détectée et le niveau de confiance
4. Testez avec des fichiers dans différentes langues (français, anglais, espagnol)

### Exercice 5: Filtrage et nettoyage

**Objectif**: Améliorer la qualité de la transcription

**Instructions**:
1. Créez un fichier `clean_transcription.py`
2. Ajoutez des fonctions pour:
   - Supprimer les doublons de mots
   - Corriger la capitalisation (première lettre des phrases)
   - Supprimer les mots de remplissage (euh, hmm, etc.)
   - Ajouter la ponctuation si manquante

{{< notice tip >}}
** Indice**: Utilisez des expressions régulières (module `re`)
{{< /notice >}}

---

## 📊 Partie 6: Analyse et comparaison

**Durée**: 20-30 minutes

### Exercice 6: Benchmark des modèles

Créez `benchmark_models.py`:

```python
#!/usr/bin/env python3
import whisper
import time

def benchmark_model(audio_path, model_name):
    """
    Teste la performance d'un modèle Whisper
    """
    print(f"\n Test du modèle: {model_name}")
    
    # Chargement
    start_load = time.time()
    model = whisper.load_model(model_name)
    load_time = time.time() - start_load
    
    # Transcription
    start_transcribe = time.time()
    result = model.transcribe(audio_path)
    transcribe_time = time.time() - start_transcribe
    
    return {
        "model": model_name,
        "load_time": load_time,
        "transcribe_time": transcribe_time,
        "total_time": load_time + transcribe_time,
        "text": result["text"],
        "language": result["language"]
    }

if __name__ == "__main__":
    import sys
    
    if len(sys.argv) < 2:
        print("Usage: python3 benchmark_models.py <fichier_audio>")
        sys.exit(1)
    
    audio_file = sys.argv[1]
    models = ["tiny", "base"]
    
    results = []
    for model_name in models:
        result = benchmark_model(audio_file, model_name)
        results.append(result)
    
    # Affichage comparatif
    print("\n" + "="*70)
    print("RÉSULTATS DU BENCHMARK")
    print("="*70)
    
    for r in results:
        print(f"\nModèle: {r['model']}")
        print(f"  Temps de chargement: {r['load_time']:.2f}s")
        print(f"  Temps de transcription: {r['transcribe_time']:.2f}s")
        print(f"  Temps total: {r['total_time']:.2f}s")
        print(f"  Langue: {r['language']}")
        print(f"  Aperçu: {r['text'][:100]}...")
```

{{< notice question >}}
**Question de réflexion**: Quel modèle choisiriez-vous pour une application en temps réel? Pourquoi?
{{< /notice >}}

---

## 🎓 Questions de compréhension

1. **Quelle est la différence entre les modèles tiny, base et large?**

2. **Pourquoi est-il important de spécifier la langue lors de la transcription?**

3. **Dans quel format sont sauvegardés les timestamps? Pourquoi le format SRT est-il utile?**

4. **Quels sont les avantages et inconvénients d'utiliser Docker pour ce type de projet?**

5. **Comment pourriez-vous améliorer la précision de la transcription pour:**
   - Un accent régional fort?
   - De la musique en arrière-plan?
   - Des termes techniques spécialisés?

---

À la fin du laboratoire, créez un fichier `RAPPORT.md` contenant:

1. **Résumé des apprentissages** (5-10 lignes)
2. **Difficultés rencontrées** et solutions
3. **Résultats des benchmarks** (tableau comparatif)
4. **Captures d'écran** des transcriptions réussies
5. **Idées d'amélioration** ou d'applications futures

---

## Pour aller plus loin

- Intégrer Whisper dans une application web (Flask/Django)
- Traiter des flux audio en temps réel
- Fine-tuner un modèle pour un vocabulaire spécifique
- Combiner ASR avec de la synthèse vocale (TTS) pour un assistant vocal
- Ajouter de la détection d'émotion dans la voix

---

## Ressources supplémentaires

- [Documentation officielle Whisper](https://github.com/openai/whisper)
- [Guide des formats audio](https://en.wikipedia.org/wiki/Audio_file_format)
- [Paper original de Whisper](https://arxiv.org/abs/2212.04356)
- [Forum de support](https://github.com/openai/whisper/discussions)

---

## Checklist de complétion

- [ ] Installation Docker réussie
- [ ] Transcription d'au moins 3 fichiers audio différents
- [ ] Tests avec 2 modèles différents (tiny et base)
- [ ] Script de transcription simple fonctionnel
- [ ] Script avancé avec timestamps fonctionnel
- [ ] Au moins 2 exercices pratiques complétés
- [ ] Benchmark réalisé
- [ ] Rapport de laboratoire rédigé

---


*N'hésitez pas à expérimenter et à poser des questions si vous rencontrez des difficultés.*


## Corrigé

- [Corrigé](./corrige/) 