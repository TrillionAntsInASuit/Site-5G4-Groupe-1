#  Corrigé complet - Laboratoire ASR avec Whisper

---

##  Table des matières

1. [Exercice 1: Transcription simple](#exercice-1)
2. [Exercice 2: Transcription avancée](#exercice-2)
3. [Exercice 3: Traitement par lot](#exercice-3)
4. [Exercice 4: Détection de langue](#exercice-4)
5. [Exercice 5: Nettoyage de transcriptions](#exercice-5)
6. [Exercice 6: Benchmark des modèles](#exercice-6)
7. [Projet final: Interface CLI](#projet-final)

---

## Exercice 1: Transcription simple {#exercice-1}

**Fichier**: `scripts/transcribe_simple.py`

```python
#!/usr/bin/env python3
"""
Exercice 1: Script de transcription simple
Transcrit un fichier audio en texte avec Whisper
"""

import whisper
import sys
import os

def transcribe_audio(audio_path, model_name="base"):
    """
    Transcrit un fichier audio en texte
    
    Args:
        audio_path (str): Chemin vers le fichier audio
        model_name (str): Nom du modèle Whisper à utiliser
    
    Returns:
        dict: Résultat de la transcription
    """
    print(f"Chargement du modèle '{model_name}'...")
    model = whisper.load_model(model_name)
    
    print(f" Transcription de: {audio_path}")
    result = model.transcribe(audio_path)
    
    return result

def main():
    """Point d'entrée principal"""
    if len(sys.argv) < 2:
        print("Usage: python3 transcribe_simple.py <fichier_audio> [modèle]")
        print("Exemple: python3 transcribe_simple.py audio.mp3 base")
        sys.exit(1)
    
    audio_file = sys.argv[1]
    model_name = sys.argv[2] if len(sys.argv) > 2 else "base"
    
    # Vérifier l'existence du fichier
    if not os.path.exists(audio_file):
        print(f" Erreur: Le fichier '{audio_file}' n'existe pas")
        sys.exit(1)
    
    # Transcription
    try:
        result = transcribe_audio(audio_file, model_name)
        
        # Affichage des résultats
        print("\n" + "="*50)
        print(" TRANSCRIPTION:")
        print("="*50)
        print(result["text"])
        print("="*50)
        print(f" Langue détectée: {result['language']}")
        print(f"  Segments: {len(result['segments'])}")
        
    except Exception as e:
        print(f" Erreur lors de la transcription: {e}")
        sys.exit(1)

if __name__ == "__main__":
    main()
```

###  Points clés de la solution:

1. **Gestion des erreurs**: Vérifie l'existence du fichier avant traitement
2. **Paramètres flexibles**: Permet de choisir le modèle en argument
3. **Affichage formaté**: Résultats clairs et lisibles
4. **Documentation**: Docstrings pour chaque fonction

---

## Exercice 2: Transcription avancée {#exercice-2}

**Fichier**: `scripts/transcribe_advanced.py`

```python
#!/usr/bin/env python3
"""
Exercice 2: Script de transcription avancée
Transcrit avec timestamps et sauvegarde multi-formats
"""

import whisper
import json
import os
from datetime import datetime

def transcribe_with_timestamps(audio_path, model_name="base", language=None):
    """
    Transcrit un fichier audio avec timestamps
    
    Args:
        audio_path (str): Chemin vers le fichier audio
        model_name (str): Nom du modèle Whisper
        language (str, optional): Code langue (fr, en, es, etc.)
    
    Returns:
        dict: Résultat complet de la transcription
    """
    print(f" Chargement du modèle '{model_name}'...")
    model = whisper.load_model(model_name)
    
    # Options de transcription
    options = {"fp16": False}  # Désactiver fp16 pour compatibilité
    if language:
        options["language"] = language
    
    print(f" Transcription en cours...")
    result = model.transcribe(audio_path, **options)
    
    return result

def save_transcription(result, audio_path, output_dir="/workspace/transcriptions"):
    """
    Sauvegarde la transcription dans différents formats
    
    Args:
        result (dict): Résultat de la transcription
        audio_path (str): Chemin du fichier audio source
        output_dir (str): Dossier de sortie
    
    Returns:
        tuple: Chemins des fichiers créés (txt, json, srt)
    """
    # Créer le dossier de sortie si nécessaire
    os.makedirs(output_dir, exist_ok=True)
    
    # Créer le nom de base pour les fichiers
    base_name = os.path.splitext(os.path.basename(audio_path))[0]
    timestamp = datetime.now().strftime("%Y%m%d_%H%M%S")
    
    # 1. Format TXT - Texte simple
    txt_path = os.path.join(output_dir, f"{base_name}_{timestamp}.txt")
    with open(txt_path, "w", encoding="utf-8") as f:
        f.write(result["text"])
    print(f" Texte sauvegardé: {txt_path}")
    
    # 2. Format JSON - Données complètes
    json_path = os.path.join(output_dir, f"{base_name}_{timestamp}.json")
    with open(json_path, "w", encoding="utf-8") as f:
        json.dump(result, f, ensure_ascii=False, indent=2)
    print(f" JSON sauvegardé: {json_path}")
    
    # 3. Format SRT - Sous-titres
    srt_path = os.path.join(output_dir, f"{base_name}_{timestamp}.srt")
    write_srt(result["segments"], srt_path)
    print(f" SRT sauvegardé: {srt_path}")
    
    # 4. Format VTT - Sous-titres web (bonus)
    vtt_path = os.path.join(output_dir, f"{base_name}_{timestamp}.vtt")
    write_vtt(result["segments"], vtt_path)
    print(f" VTT sauvegardé: {vtt_path}")
    
    return txt_path, json_path, srt_path, vtt_path

def write_srt(segments, output_path):
    """
    Écrit un fichier SRT (SubRip) pour sous-titres
    
    Args:
        segments (list): Liste des segments avec timestamps
        output_path (str): Chemin du fichier de sortie
    """
    with open(output_path, "w", encoding="utf-8") as f:
        for i, segment in enumerate(segments, start=1):
            start_time = format_timestamp_srt(segment["start"])
            end_time = format_timestamp_srt(segment["end"])
            text = segment["text"].strip()
            
            f.write(f"{i}\n")
            f.write(f"{start_time} --> {end_time}\n")
            f.write(f"{text}\n\n")

def write_vtt(segments, output_path):
    """
    Écrit un fichier VTT (WebVTT) pour sous-titres web
    
    Args:
        segments (list): Liste des segments avec timestamps
        output_path (str): Chemin du fichier de sortie
    """
    with open(output_path, "w", encoding="utf-8") as f:
        f.write("WEBVTT\n\n")
        
        for i, segment in enumerate(segments, start=1):
            start_time = format_timestamp_vtt(segment["start"])
            end_time = format_timestamp_vtt(segment["end"])
            text = segment["text"].strip()
            
            f.write(f"{i}\n")
            f.write(f"{start_time} --> {end_time}\n")
            f.write(f"{text}\n\n")

def format_timestamp_srt(seconds):
    """
    Formate les secondes en format SRT (HH:MM:SS,mmm)
    
    Args:
        seconds (float): Temps en secondes
    
    Returns:
        str: Timestamp formaté
    """
    hours = int(seconds // 3600)
    minutes = int((seconds % 3600) // 60)
    secs = int(seconds % 60)
    millis = int((seconds % 1) * 1000)
    return f"{hours:02d}:{minutes:02d}:{secs:02d},{millis:03d}"

def format_timestamp_vtt(seconds):
    """
    Formate les secondes en format VTT (HH:MM:SS.mmm)
    
    Args:
        seconds (float): Temps en secondes
    
    Returns:
        str: Timestamp formaté
    """
    hours = int(seconds // 3600)
    minutes = int((seconds % 3600) // 60)
    secs = int(seconds % 60)
    millis = int((seconds % 1) * 1000)
    return f"{hours:02d}:{minutes:02d}:{secs:02d}.{millis:03d}"

def display_segments(segments):
    """
    Affiche les segments avec timestamps dans le terminal
    
    Args:
        segments (list): Liste des segments
    """
    print("\n" + "="*70)
    print(" TRANSCRIPTION AVEC TIMESTAMPS:")
    print("="*70)
    
    for segment in segments:
        start = segment["start"]
        end = segment["end"]
        text = segment["text"].strip()
        duration = end - start
        print(f"[{start:>6.2f}s → {end:>6.2f}s] ({duration:>5.2f}s) {text}")
    
    print("="*70)

def display_statistics(result):
    """
    Affiche des statistiques sur la transcription
    
    Args:
        result (dict): Résultat de la transcription
    """
    segments = result["segments"]
    total_duration = segments[-1]["end"] if segments else 0
    word_count = len(result["text"].split())
    
    print("\n" + "="*70)
    print(" STATISTIQUES:")
    print("="*70)
    print(f" Langue détectée: {result['language']}")
    print(f"  Durée totale: {total_duration:.2f} secondes ({total_duration/60:.2f} minutes)")
    print(f" Nombre de segments: {len(segments)}")
    print(f" Nombre de mots: {word_count}")
    print(f"Vitesse moyenne: {word_count / (total_duration / 60):.1f} mots/minute")
    print("="*70)

def main():
    """Point d'entrée principal"""
    import sys
    
    if len(sys.argv) < 2:
        print("Usage: python3 transcribe_advanced.py <fichier_audio> [modèle] [langue]")
        print("\nExemples:")
        print("  python3 transcribe_advanced.py audio.mp3")
        print("  python3 transcribe_advanced.py audio.mp3 base")
        print("  python3 transcribe_advanced.py audio.mp3 base fr")
        print("\nModèles disponibles: tiny, base, small, medium, large")
        print("Langues courantes: fr, en, es, de, it, pt, etc.")
        sys.exit(1)
    
    audio_file = sys.argv[1]
    model_name = sys.argv[2] if len(sys.argv) > 2 else "base"
    language = sys.argv[3] if len(sys.argv) > 3 else None
    
    # Vérifier l'existence du fichier
    if not os.path.exists(audio_file):
        print(f" Erreur: Le fichier '{audio_file}' n'existe pas")
        sys.exit(1)
    
    try:
        # Transcription
        result = transcribe_with_timestamps(audio_file, model_name, language)
        
        # Affichage
        display_statistics(result)
        display_segments(result["segments"])
        
        # Sauvegarde
        print("\n Sauvegarde des résultats...")
        save_transcription(result, audio_file)
        
        print("\n Transcription terminée avec succès!")
        
    except Exception as e:
        print(f" Erreur: {e}")
        import traceback
        traceback.print_exc()
        sys.exit(1)

if __name__ == "__main__":
    main()
```

###  Points clés de la solution:

1. **Multi-formats**: TXT, JSON, SRT, VTT
2. **Statistiques**: Durée, nombre de mots, vitesse
3. **Timestamps précis**: Formats SRT et VTT différents
4. **Gestion complète des erreurs**

---

## Exercice 3: Traitement par lot {#exercice-3}

**Fichier**: `solutions/batch_transcribe.py`

```python
#!/usr/bin/env python3
"""
Exercice 3: Traitement par lot (Batch Processing)
Transcrit automatiquement tous les fichiers audio d'un dossier
"""

import whisper
import os
import json
from datetime import datetime
import time

# Extensions audio supportées
AUDIO_EXTENSIONS = {'.mp3', '.wav', '.m4a', '.flac', '.ogg', '.opus', '.webm'}

def find_audio_files(directory):
    """
    Trouve tous les fichiers audio dans un dossier
    
    Args:
        directory (str): Chemin du dossier à scanner
    
    Returns:
        list: Liste des chemins des fichiers audio trouvés
    """
    audio_files = []
    
    if not os.path.exists(directory):
        print(f" Le dossier '{directory}' n'existe pas")
        return audio_files
    
    for filename in os.listdir(directory):
        file_path = os.path.join(directory, filename)
        
        # Vérifier que c'est un fichier et qu'il a une extension audio
        if os.path.isfile(file_path):
            _, ext = os.path.splitext(filename)
            if ext.lower() in AUDIO_EXTENSIONS:
                audio_files.append(file_path)
    
    return sorted(audio_files)

def transcribe_file(model, audio_path, language=None):
    """
    Transcrit un seul fichier avec gestion d'erreur
    
    Args:
        model: Modèle Whisper chargé
        audio_path (str): Chemin du fichier
        language (str, optional): Code langue
    
    Returns:
        dict: Résultat ou None en cas d'erreur
    """
    try:
        options = {"fp16": False}
        if language:
            options["language"] = language
        
        result = model.transcribe(audio_path, **options)
        return result
    except Exception as e:
        print(f" Erreur lors de la transcription de {audio_path}: {e}")
        return None

def save_batch_result(result, audio_path, output_dir):
    """
    Sauvegarde un résultat de transcription
    
    Args:
        result (dict): Résultat de la transcription
        audio_path (str): Fichier source
        output_dir (str): Dossier de sortie
    
    Returns:
        bool: True si succès, False sinon
    """
    try:
        os.makedirs(output_dir, exist_ok=True)
        
        base_name = os.path.splitext(os.path.basename(audio_path))[0]
        
        # Sauvegarder le texte
        txt_path = os.path.join(output_dir, f"{base_name}.txt")
        with open(txt_path, "w", encoding="utf-8") as f:
            f.write(result["text"])
        
        # Sauvegarder le JSON complet
        json_path = os.path.join(output_dir, f"{base_name}.json")
        with open(json_path, "w", encoding="utf-8") as f:
            json.dump(result, f, ensure_ascii=False, indent=2)
        
        return True
    except Exception as e:
        print(f" Erreur de sauvegarde pour {audio_path}: {e}")
        return False

def generate_report(results, output_dir):
    """
    Génère un rapport récapitulatif du traitement par lot
    
    Args:
        results (list): Liste des résultats
        output_dir (str): Dossier de sortie
    """
    report_path = os.path.join(output_dir, f"batch_report_{datetime.now().strftime('%Y%m%d_%H%M%S')}.txt")
    
    total_files = len(results)
    successful = sum(1 for r in results if r['success'])
    failed = total_files - successful
    total_duration = sum(r['duration'] for r in results if r['success'])
    total_words = sum(r['word_count'] for r in results if r['success'])
    
    with open(report_path, "w", encoding="utf-8") as f:
        f.write("="*70 + "\n")
        f.write(" RAPPORT DE TRAITEMENT PAR LOT\n")
        f.write("="*70 + "\n\n")
        
        f.write(f" Date: {datetime.now().strftime('%Y-%m-%d %H:%M:%S')}\n\n")
        
        f.write(" STATISTIQUES GLOBALES\n")
        f.write("-" * 70 + "\n")
        f.write(f" Fichiers traités: {total_files}\n")
        f.write(f" Réussis: {successful}\n")
        f.write(f" Échoués: {failed}\n")
        f.write(f"  Durée audio totale: {total_duration:.2f}s ({total_duration/60:.2f} min)\n")
        f.write(f" Mots transcrits: {total_words}\n")
        f.write(f"⚡ Vitesse moyenne: {total_words / (total_duration / 60) if total_duration > 0 else 0:.1f} mots/min\n\n")
        
        f.write(" DÉTAILS PAR FICHIER\n")
        f.write("-" * 70 + "\n\n")
        
        for i, result in enumerate(results, 1):
            f.write(f"{i}. {os.path.basename(result['file'])}\n")
            f.write(f"   Statut: {' Réussi' if result['success'] else '❌ Échoué'}\n")
            
            if result['success']:
                f.write(f"   Langue: {result['language']}\n")
                f.write(f"   Durée: {result['duration']:.2f}s\n")
                f.write(f"   Mots: {result['word_count']}\n")
                f.write(f"   Temps de traitement: {result['processing_time']:.2f}s\n")
                f.write(f"   Aperçu: {result['text'][:100]}...\n")
            else:
                f.write(f"   Erreur: {result['error']}\n")
            
            f.write("\n")
        
        f.write("="*70 + "\n")
    
    print(f"\n Rapport généré: {report_path}")
    return report_path

def main():
    """Point d'entrée principal"""
    import sys
    
    if len(sys.argv) < 2:
        print("Usage: python3 batch_transcribe.py <dossier_audio> [modèle] [langue]")
        print("\nExemples:")
        print("  python3 batch_transcribe.py /workspace/audio_samples")
        print("  python3 batch_transcribe.py /workspace/audio_samples base")
        print("  python3 batch_transcribe.py /workspace/audio_samples base fr")
        sys.exit(1)
    
    input_dir = sys.argv[1]
    model_name = sys.argv[2] if len(sys.argv) > 2 else "base"
    language = sys.argv[3] if len(sys.argv) > 3 else None
    output_dir = "/workspace/transcriptions/batch"
    
    # Trouver les fichiers audio
    print(f" Recherche de fichiers audio dans: {input_dir}")
    audio_files = find_audio_files(input_dir)
    
    if not audio_files:
        print(f" Aucun fichier audio trouvé dans '{input_dir}'")
        print(f" Extensions supportées: {', '.join(AUDIO_EXTENSIONS)}")
        sys.exit(1)
    
    print(f" {len(audio_files)} fichier(s) trouvé(s)")
    for i, f in enumerate(audio_files, 1):
        print(f"  {i}. {os.path.basename(f)}")
    
    # Charger le modèle une seule fois
    print(f"\n Chargement du modèle '{model_name}'...")
    try:
        model = whisper.load_model(model_name)
    except Exception as e:
        print(f" Erreur de chargement du modèle: {e}")
        sys.exit(1)
    
    # Traiter chaque fichier
    print(f"\n🎤 Début du traitement par lot...")
    results = []
    
    for i, audio_file in enumerate(audio_files, 1):
        print(f"\n[{i}/{len(audio_files)}] 🎵 {os.path.basename(audio_file)}")
        
        start_time = time.time()
        result = transcribe_file(model, audio_file, language)
        processing_time = time.time() - start_time
        
        if result:
            # Sauvegarder
            success = save_batch_result(result, audio_file, output_dir)
            
            # Calculer les statistiques
            word_count = len(result["text"].split())
            duration = result["segments"][-1]["end"] if result["segments"] else 0
            
            results.append({
                'file': audio_file,
                'success': success,
                'language': result['language'],
                'duration': duration,
                'word_count': word_count,
                'processing_time': processing_time,
                'text': result['text']
            })
            
            print(f"   Transcrit en {processing_time:.2f}s")
            print(f"    {word_count} mots, durée audio: {duration:.2f}s")
        else:
            results.append({
                'file': audio_file,
                'success': False,
                'error': 'Échec de transcription'
            })
    
    # Générer le rapport
    print("\n Génération du rapport...")
    generate_report(results, output_dir)
    
    # Résumé final
    successful = sum(1 for r in results if r['success'])
    print("\n" + "="*70)
    print(" TRAITEMENT PAR LOT TERMINÉ")
    print("="*70)
    print(f" Fichiers transcrits avec succès: {successful}/{len(audio_files)}")
    print(f" Résultats sauvegardés dans: {output_dir}")
    print("="*70)

if __name__ == "__main__":
    main()
```

###  Points clés de la solution:

1. **Détection automatique**: Trouve tous les fichiers audio
2. **Optimisation**: Charge le modèle une seule fois
3. **Gestion d'erreurs robuste**: Continue même si un fichier échoue
4. **Rapport détaillé**: Statistiques complètes
5. **Support multi-formats**: MP3, WAV, FLAC, etc.

---

## Exercice 4: Détection de langue {#exercice-4}

**Fichier**: `solutions/detect_language.py`

```python
#!/usr/bin/env python3
"""
Exercice 4: Détection automatique de langue
Identifie la langue parlée dans un fichier audio
"""

import whisper
import sys
import os
import json
from collections import Counter

# Noms complets des langues
LANGUAGE_NAMES = {
    'en': 'Anglais',
    'fr': 'Français',
    'es': 'Espagnol',
    'de': 'Allemand',
    'it': 'Italien',
    'pt': 'Portugais',
    'nl': 'Néerlandais',
    'ru': 'Russe',
    'zh': 'Chinois',
    'ja': 'Japonais',
    'ko': 'Coréen',
    'ar': 'Arabe',
    'hi': 'Hindi',
    'tr': 'Turc',
    'pl': 'Polonais',
    'uk': 'Ukrainien',
    'vi': 'Vietnamien',
    'sv': 'Suédois',
    'no': 'Norvégien',
    'da': 'Danois',
    'fi': 'Finnois',
    'cs': 'Tchèque',
    'ro': 'Roumain',
    'el': 'Grec',
    'hu': 'Hongrois',
    'th': 'Thaï',
    'id': 'Indonésien',
    'ms': 'Malais',
    'he': 'Hébreu',
    'fa': 'Persan',
    'bn': 'Bengali',
    'ta': 'Tamoul',
    'te': 'Télougou',
    'mr': 'Marathi',
    'ur': 'Ourdou',
    'gu': 'Gujarati',
    'kn': 'Kannada',
    'ml': 'Malayalam',
    'pa': 'Pendjabi',
    'sw': 'Swahili',
    'af': 'Afrikaans',
    'is': 'Islandais',
    'et': 'Estonien',
    'lv': 'Letton',
    'lt': 'Lituanien',
    'sl': 'Slovène',
    'sk': 'Slovaque',
    'hr': 'Croate',
    'sr': 'Serbe',
    'bg': 'Bulgare',
    'mk': 'Macédonien',
    'sq': 'Albanais',
    'ca': 'Catalan',
    'eu': 'Basque',
    'gl': 'Galicien',
    'cy': 'Gallois',
    'ga': 'Irlandais',
    'mt': 'Maltais',
}

def detect_language_whisper(audio_path, model_name="base"):
    """
    Détecte la langue en utilisant Whisper
    
    Args:
        audio_path (str): Chemin du fichier audio
        model_name (str): Nom du modèle Whisper
    
    Returns:
        dict: Informations sur la langue détectée
    """
    print(f" Chargement du modèle '{model_name}'...")
    model = whisper.load_model(model_name)
    
    # Charger l'audio
    print(f" Chargement de l'audio...")
    audio = whisper.load_audio(audio_path)
    audio = whisper.pad_or_trim(audio)
    
    # Créer le mel spectrogram
    mel = whisper.log_mel_spectrogram(audio).to(model.device)
    
    # Détecter la langue
    print(f" Détection de la langue...")
    _, probs = model.detect_language(mel)
    
    # Obtenir les langues les plus probables
    top_languages = sorted(probs.items(), key=lambda x: x[1], reverse=True)[:10]
    
    detected_lang = top_languages[0][0]
    confidence = top_languages[0][1]
    
    return {
        'detected': detected_lang,
        'confidence': confidence,
        'top_languages': top_languages,
        'all_probs': probs
    }

def transcribe_with_language(audio_path, model_name="base"):
    """
    Transcrit en laissant Whisper détecter la langue automatiquement
    
    Args:
        audio_path (str): Chemin du fichier audio
        model_name (str): Nom du modèle
    
    Returns:
        dict: Résultat de transcription avec langue détectée
    """
    print(f" Chargement du modèle '{model_name}'...")
    model = whisper.load_model(model_name)
    
    print(f" Transcription avec détection automatique...")
    result = model.transcribe(audio_path, fp16=False)
    
    return result

def display_language_info(lang_info):
    """
    Affiche les informations de détection de langue
    
    Args:
        lang_info (dict): Informations de langue
    """
    detected = lang_info['detected']
    confidence = lang_info['confidence']
    lang_name = LANGUAGE_NAMES.get(detected, detected.upper())
    
    print("\n" + "="*70)
    print(" DÉTECTION DE LANGUE")
    print("="*70)
    print(f"Langue détectée: {lang_name} ({detected})")
    print(f" Confiance: {confidence*100:.2f}%")
    
    # Barre de confiance visuelle
    bar_length = 50
    filled = int(bar_length * confidence)
    bar = "█" * filled + "░" * (bar_length - filled)
    print(f" {bar} {confidence*100:.1f}%")
    
    print("\n Top 10 des langues détectées:")
    print("-" * 70)
    
    for i, (lang, prob) in enumerate(lang_info['top_languages'], 1):
        name = LANGUAGE_NAMES.get(lang, lang.upper())
        bar_len = 30
        filled_bar = int(bar_len * prob)
        bar = "█" * filled_bar + "░" * (bar_len - filled_bar)
        print(f"{i:2d}. {name:20s} ({lang}) {bar} {prob*100:5.2f}%")
    
    print("="*70)

def compare_multiple_files(audio_files, model_name="base"):
    """
    Compare la langue de plusieurs fichiers audio
    
    Args:
        audio_files (list): Liste de chemins de fichiers
        model_name (str): Nom du modèle
    
    Returns:
        list: Résultats pour chaque fichier
    """
    print(f" Chargement du modèle '{model_name}'...")
    model = whisper.load_model(model_name)
    
    results = []
    
    for i, audio_path in enumerate(audio_files, 1):
        print(f"\n[{i}/{len(audio_files)}] 🎵 {os.path.basename(audio_path)}")
        
        try:
            audio = whisper.load_audio(audio_path)
            audio = whisper.pad_or_trim(audio)
            mel = whisper.log_mel_spectrogram(audio).to(model.device)
            _, probs = model.detect_language(mel)
            
            detected = max(probs, key=probs.get)
            confidence = probs[detected]
            
            results.append({
                'file': audio_path,
                'language': detected,
                'confidence': confidence,
                'probs': probs
            })
            
            lang_name = LANGUAGE_NAMES.get(detected, detected.upper())
            print(f"    {lang_name} ({detected}) - {confidence*100:.1f}%")
            
        except Exception as e:
            print(f"   ❌ Erreur: {e}")
            results.append({
                'file': audio_path,
                'error': str(e)
            })
    
    return results

def save_language_report(results, output_path):
    """
    Sauvegarde un rapport de détection de langue
    
    Args:
        results (list): Résultats de détection
        output_path (str): Chemin du fichier de sortie
    """
    with open(output_path, 'w', encoding='utf-8') as f:
        json.dump(results, f, ensure_ascii=False, indent=2)
    
    print(f"\n Rapport sauvegardé: {output_path}")

def main():
    """Point d'entrée principal"""
    if len(sys.argv) < 2:
        print("Usage: python3 detect_language.py <fichier(s)_audio> [modèle]")
        print("\nExemples:")
        print("  python3 detect_language.py audio.mp3")
        print("  python3 detect_language.py audio1.mp3 audio2.mp3 audio3.mp3")
        print("  python3 detect_language.py audio.mp3 base")
        print("\nModes:")
        print("  1 fichier  : Détection détaillée avec top 10")
        print("  N fichiers : Comparaison rapide")
        sys.exit(1)
    
    # Séparer les fichiers du modèle
    args = sys.argv[1:]
    model_name = "base"
    
    # Si le dernier argument est un modèle connu
    if args[-1] in ['tiny', 'base', 'small', 'medium', 'large']:
        model_name = args[-1]
        audio_files = args[:-1]
    else:
        audio_files = args
    
    # Vérifier l'existence des fichiers
    for f in audio_files:
        if not os.path.exists(f):
            print(f" Fichier non trouvé: {f}")
            sys.exit(1)
    
    try:
        if len(audio_files) == 1:
            # Mode détaillé pour un seul fichier
            print(f"🎵 Analyse de: {os.path.basename(audio_files[0])}")
            
            # Détection détaillée
            lang_info = detect_language_whisper(audio_files[0], model_name)
            display_language_info(lang_info)
            
            # Transcription avec langue détectée
            print(f"\n Transcription avec la langue détectée...")
            result = transcribe_with_language(audio_files[0], model_name)
            
            print("\n TRANSCRIPTION:")
            print("-" * 70)
            print(result['text'])
            print("-" * 70)
            
            # Sauvegarder
            output_path = f"language_detection_{os.path.splitext(os.path.basename(audio_files[0]))[0]}.json"
            with open(output_path, 'w', encoding='utf-8') as f:
                json.dump({
                    'file': audio_files[0],
                    'detection': lang_info,
                    'transcription': result
                }, f, ensure_ascii=False, indent=2)
            
            print(f"\n Résultats sauvegardés: {output_path}")
            
        else:
            # Mode comparaison pour plusieurs fichiers
            print(f"🎵 Analyse de {len(audio_files)} fichiers...")
            results = compare_multiple_files(audio_files, model_name)
            
            # Afficher résumé
            print("\n" + "="*70)
            print(" RÉSUMÉ DES LANGUES DÉTECTÉES")
            print("="*70)
            
            languages = [r['language'] for r in results if 'language' in r]
            lang_counts = Counter(languages)
            
            for lang, count in lang_counts.most_common():
                lang_name = LANGUAGE_NAMES.get(lang, lang.upper())
                print(f" {lang_name:20s} ({lang}): {count} fichier(s)")
            
            # Sauvegarder
            output_path = "language_detection_batch.json"
            save_language_report(results, output_path)
            
        print("\n✨ Détection terminée avec succès!")
        
    except Exception as e:
        print(f" Erreur: {e}")
        import traceback
        traceback.print_exc()
        sys.exit(1)

if __name__ == "__main__":
    main()
```

###  Points clés de la solution:

1. **Détection native Whisper**: Utilise l'API de détection
2. **Confiance visuelle**: Barres de progression
3. **Top 10 langues**: Probabilités pour chaque langue
4. **Mode batch**: Compare plusieurs fichiers
5. **Dictionnaire multilingue**: 50+ langues supportées

---

## Exercice 5: Nettoyage de transcriptions {#exercice-5}

**Fichier**: `solutions/clean_transcription.py`

```python
#!/usr/bin/env python3
"""
Exercice 5: Nettoyage et amélioration de transcriptions
Améliore la qualité du texte transcrit
"""

import whisper
import re
import sys
import os
from collections import Counter

# Mots de remplissage courants en français
FILLER_WORDS_FR = {
    'euh', 'heu', 'euhm', 'hem', 'hum', 'hmm', 'hm',
    'ben', 'bah', 'pfff', 'pff', 'tss',
    'genre', 'style', 'quoi', 'voilà', 'donc',
}

# Mots de remplissage en anglais
FILLER_WORDS_EN = {
    'uh', 'um', 'umm', 'err', 'ah', 'eh', 'oh',
    'like', 'you know', 'i mean', 'kind of', 'sort of',
    'basically', 'actually', 'literally',
}

def remove_filler_words(text, language='fr'):
    """
    Supprime les mots de remplissage
    
    Args:
        text (str): Texte à nettoyer
        language (str): Langue du texte
    
    Returns:
        str: Texte nettoyé
    """
    fillers = FILLER_WORDS_FR if language == 'fr' else FILLER_WORDS_EN
    
    # Pattern pour matcher les mots de remplissage
    pattern = r'\b(' + '|'.join(re.escape(f) for f in fillers) + r')\b'
    
    # Supprimer les mots de remplissage (insensible à la casse)
    cleaned = re.sub(pattern, '', text, flags=re.IGNORECASE)
    
    # Nettoyer les espaces multiples
    cleaned = re.sub(r'\s+', ' ', cleaned)
    
    return cleaned.strip()

def remove_repeated_words(text):
    """
    Supprime les répétitions de mots consécutifs
    Ex: "je je veux" -> "je veux"
    
    Args:
        text (str): Texte à nettoyer
    
    Returns:
        str: Texte sans répétitions
    """
    # Pattern pour détecter les mots répétés
    pattern = r'\b(\w+)(\s+\1\b)+'
    
    cleaned = re.sub(pattern, r'\1', text, flags=re.IGNORECASE)
    
    return cleaned

def fix_capitalization(text):
    """
    Corrige la capitalisation (majuscules en début de phrase)
    
    Args:
        text (str): Texte à corriger
    
    Returns:
        str: Texte avec capitalisation correcte
    """
    # Mettre tout en minuscules d'abord
    text = text.lower()
    
    # Majuscule au début du texte
    if text:
        text = text[0].upper() + text[1:]
    
    # Majuscule après les points
    text = re.sub(r'([.!?]\s+)([a-z])', lambda m: m.group(1) + m.group(2).upper(), text)
    
    # Majuscule pour "je" en français
    text = re.sub(r'\bje\b', 'Je', text)
    
    # Majuscule pour "i" en anglais
    text = re.sub(r'\bi\b', 'I', text)
    
    return text

def fix_punctuation(text):
    """
    Améliore la ponctuation
    
    Args:
        text (str): Texte à corriger
    
    Returns:
        str: Texte avec ponctuation améliorée
    """
    # Ajouter un point à la fin si manquant
    if text and text[-1] not in '.!?':
        text = text + '.'
    
    # Corriger les espaces avant la ponctuation
    text = re.sub(r'\s+([,.!?;:])', r'\1', text)
    
    # Ajouter un espace après la ponctuation si manquant
    text = re.sub(r'([,.!?;:])([A-Za-z])', r'\1 \2', text)
    
    # Supprimer les espaces multiples
    text = re.sub(r'\s+', ' ', text)
    
    # Corriger les points multiples
    text = re.sub(r'\.{2,}', '.', text)
    
    return text.strip()

def remove_excessive_spaces(text):
    """
    Supprime les espaces excessifs
    
    Args:
        text (str): Texte à nettoyer
    
    Returns:
        str: Texte nettoyé
    """
    # Remplacer espaces multiples par un seul
    text = re.sub(r'\s+', ' ', text)
    
    # Supprimer espaces en début et fin
    text = text.strip()
    
    return text

def clean_transcription(text, language='fr', options=None):
    """
    Nettoie une transcription complète
    
    Args:
        text (str): Texte à nettoyer
        language (str): Langue du texte
        options (dict): Options de nettoyage
    
    Returns:
        dict: Texte nettoyé et statistiques
    """
    if options is None:
        options = {
            'remove_fillers': True,
            'remove_repetitions': True,
            'fix_capitalization': True,
            'fix_punctuation': True,
        }
    
    original_text = text
    stats = {
        'original_length': len(text),
        'original_words': len(text.split()),
    }
    
    # Étape 1: Supprimer les mots de remplissage
    if options.get('remove_fillers', True):
        text = remove_filler_words(text, language)
        stats['fillers_removed'] = stats['original_words'] - len(text.split())
    
    # Étape 2: Supprimer les répétitions
    if options.get('remove_repetitions', True):
        text = remove_repeated_words(text)
    
    # Étape 3: Corriger la capitalisation
    if options.get('fix_capitalization', True):
        text = fix_capitalization(text)
    
    # Étape 4: Corriger la ponctuation
    if options.get('fix_punctuation', True):
        text = fix_punctuation(text)
    
    # Étape 5: Nettoyer les espaces
    text = remove_excessive_spaces(text)
    
    stats['cleaned_length'] = len(text)
    stats['cleaned_words'] = len(text.split())
    stats['reduction_percent'] = ((stats['original_length'] - stats['cleaned_length']) / stats['original_length'] * 100) if stats['original_length'] > 0 else 0
    
    return {
        'original': original_text,
        'cleaned': text,
        'stats': stats
    }

def transcribe_and_clean(audio_path, model_name="base", language=None, clean_options=None):
    """
    Transcrit et nettoie un fichier audio
    
    Args:
        audio_path (str): Chemin du fichier audio
        model_name (str): Modèle Whisper
        language (str): Langue (détection auto si None)
        clean_options (dict): Options de nettoyage
    
    Returns:
        dict: Résultats complets
    """
    print(f" Chargement du modèle '{model_name}'...")
    model = whisper.load_model(model_name)
    
    print(f" Transcription de: {os.path.basename(audio_path)}")
    
    options = {"fp16": False}
    if language:
        options["language"] = language
    
    result = model.transcribe(audio_path, **options)
    
    detected_lang = result['language']
    original_text = result['text']
    
    print(f" Langue détectée: {detected_lang}")
    print(f"\n Nettoyage du texte...")
    
    cleaning_result = clean_transcription(original_text, detected_lang, clean_options)
    
    return {
        'audio_file': audio_path,
        'language': detected_lang,
        'transcription_raw': result,
        'cleaning': cleaning_result
    }

def display_results(results):
    """
    Affiche les résultats du nettoyage
    
    Args:
        results (dict): Résultats du traitement
    """
    print("\n" + "="*70)
    print(" TRANSCRIPTION ORIGINALE:")
    print("="*70)
    print(results['cleaning']['original'])
    
    print("\n" + "="*70)
    print(" TRANSCRIPTION NETTOYÉE:")
    print("="*70)
    print(results['cleaning']['cleaned'])
    
    stats = results['cleaning']['stats']
    print("\n" + "="*70)
    print(" STATISTIQUES DU NETTOYAGE:")
    print("="*70)
    print(f" Longueur originale: {stats['original_length']} caractères, {stats['original_words']} mots")
    print(f"  Longueur nettoyée: {stats['cleaned_length']} caractères, {stats['cleaned_words']} mots")
    print(f" Réduction: {stats['reduction_percent']:.1f}%")
    if 'fillers_removed' in stats:
        print(f"  Mots de remplissage retirés: {stats['fillers_removed']}")
    print("="*70)

def save_cleaned_transcription(results, output_dir="/workspace/transcriptions/cleaned"):
    """
    Sauvegarde la transcription nettoyée
    
    Args:
        results (dict): Résultats du traitement
        output_dir (str): Dossier de sortie
    """
    os.makedirs(output_dir, exist_ok=True)
    
    base_name = os.path.splitext(os.path.basename(results['audio_file']))[0]
    
    # Sauvegarder le texte original
    original_path = os.path.join(output_dir, f"{base_name}_original.txt")
    with open(original_path, 'w', encoding='utf-8') as f:
        f.write(results['cleaning']['original'])
    
    # Sauvegarder le texte nettoyé
    cleaned_path = os.path.join(output_dir, f"{base_name}_cleaned.txt")
    with open(cleaned_path, 'w', encoding='utf-8') as f:
        f.write(results['cleaning']['cleaned'])
    
    # Sauvegarder le rapport complet
    report_path = os.path.join(output_dir, f"{base_name}_report.json")
    import json
    with open(report_path, 'w', encoding='utf-8') as f:
        json.dump(results, f, ensure_ascii=False, indent=2)
    
    print(f"\n Fichiers sauvegardés:")
    print(f"    Original: {original_path}")
    print(f"    Nettoyé: {cleaned_path}")
    print(f"    Rapport: {report_path}")

def main():
    """Point d'entrée principal"""
    if len(sys.argv) < 2:
        print("Usage: python3 clean_transcription.py <fichier_audio> [modèle] [langue]")
        print("\nExemples:")
        print("  python3 clean_transcription.py audio.mp3")
        print("  python3 clean_transcription.py audio.mp3 base")
        print("  python3 clean_transcription.py audio.mp3 base fr")
        print("\nOptions de nettoyage disponibles:")
        print("  - Suppression des mots de remplissage (euh, hmm, etc.)")
        print("  - Suppression des répétitions de mots")
        print("  - Correction de la capitalisation")
        print("  - Amélioration de la ponctuation")
        sys.exit(1)
    
    audio_file = sys.argv[1]
    model_name = sys.argv[2] if len(sys.argv) > 2 else "base"
    language = sys.argv[3] if len(sys.argv) > 3 else None
    
    if not os.path.exists(audio_file):
        print(f" Fichier non trouvé: {audio_file}")
        sys.exit(1)
    
    try:
        # Transcription et nettoyage
        results = transcribe_and_clean(audio_file, model_name, language)
        
        # Affichage
        display_results(results)
        
        # Sauvegarde
        save_cleaned_transcription(results)
        
        print("\n✨ Nettoyage terminé avec succès!")
        
    except Exception as e:
        print(f" Erreur: {e}")
        import traceback
        traceback.print_exc()
        sys.exit(1)

if __name__ == "__main__":
    main()
```

###  Points clés de la solution:

1. **Filtres multiples**: Mots de remplissage, répétitions
2. **Correction automatique**: Capitalisation, ponctuation
3. **Statistiques**: Mesure de l'amélioration
4. **Multilingue**: Support FR/EN
5. **Configurable**: Options de nettoyage personnalisables

---