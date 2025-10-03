# helfer_bot.py
# Dieser Bot sucht auf GitHub nach relevanten Konversationen und erstellt Issues im eigenen Repository.

import os
import requests
from datetime import datetime, timedelta

# --- KONFIGURATION ---
# Angepasst an dein persönliches GitHub-Projekt.
OWNER = "dennispriegnitz43-arch"
REPO = "Quantum-Leap-API"

# Die Schlüsselwörter, nach denen der Bot suchen soll.
KEYWORDS = [
    "cheap image api",
    "stable diffusion api alternative",
    "pay-as-you-go image generation",
    "dall-e api too expensive",
    "affordable image generation api"
]

# GitHub API-Token wird sicher von der Action-Umgebung geladen.
GITHUB_TOKEN = os.getenv("GITHUB_TOKEN")
HEADERS = {"Authorization": f"token {GITHUB_TOKEN}"}

def search_github_for_opportunities():
    """Durchsucht GitHub nach neuen Issues, die zu den Keywords passen."""
    print("Starte die Suche nach neuen Chancen...")
    
    since_date = (datetime.utcnow() - timedelta(days=1)).strftime('%Y-%m-%dT%H:%M:%SZ')
    found_links = set()

    for keyword in KEYWORDS:
        query = f'"{keyword}" in:title,body is:issue is:open created:>{since_date}'
        search_url = f"https://api.github.com/search/issues?q={query}&sort=created&order=desc"
        
        try:
            response = requests.get(search_url, headers=HEADERS)
            response.raise_for_status()
            results = response.json().get("items", [])
            print(f"Suche für '{keyword}': {len(results)} Ergebnis(se) gefunden.")
            for item in results:
                found_links.add(item["html_url"])
        except requests.exceptions.RequestException as e:
            print(f"Fehler bei der API-Anfrage für Keyword '{keyword}': {e}")
            continue
    return list(found_links)

def check_if_issue_exists(link):
    """Prüft, ob für diesen Link bereits ein Issue in unserem Repo existiert."""
    search_query = f'"{link}" in:body repo:{OWNER}/{REPO} is:issue'
    search_url = f"https://api.github.com/search/issues?q={search_query}"
    
    try:
        response = requests.get(search_url, headers=HEADERS)
        response.raise_for_status()
        return response.json().get("total_count", 0) > 0
    except requests.exceptions.RequestException as e:
        print(f"Fehler bei der Prüfung, ob Issue existiert: {e}")
        return True

def create_issue_for_opportunity(link):
    """Erstellt ein neues Issue in unserem eigenen Repository, um uns zu benachrichtigen."""
    if check_if_issue_exists(link):
        print(f"Info: Issue für {link} existiert bereits. Überspringe.")
        return

    issue_title = f"Neue Chance gefunden: {link.split('/')[-4]}/{link.split('/')[-3]}"
    issue_body = f"**Chance entdeckt!**\n\nEine potenziell relevante Konversation wurde gefunden:\n\n**Link:** {link}\n\nBitte manuell prüfen und ggf. eine hilfreiche, persönliche Antwort verfassen."
    issue_url = f"https://api.github.com/repos/{OWNER}/{REPO}/issues"
    
    try:
        response = requests.post(issue_url, headers=HEADERS, json={"title": issue_title, "body": issue_body})
        response.raise_for_status()
        print(f"Erfolgreich Issue erstellt für: {link}")
    except requests.exceptions.RequestException as e:
        print(f"Fehler beim Erstellen des Issues: {e.response.text}")

if __name__ == "__main__":
    if not GITHUB_TOKEN:
        raise Exception("Fehler: GITHUB_TOKEN nicht gefunden. Dieses Skript muss als GitHub Action laufen.")
    
    opportunities = search_github_for_opportunities()
    
    if not opportunities:
        print("Keine neuen Chancen in den letzten 24 Stunden gefunden.")
    else:
        print(f"\n{len(opportunities)} einzigartige Chance(n) gefunden. Erstelle jetzt Benachrichtigungs-Issues...")
        for opp_link in opportunities:
            create_issue_for_opportunity(opp_link)
    
    print("\nHelfer-Bot-Lauf abgeschlossen.")
