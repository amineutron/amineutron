**English summary.** I build local-first AI assistants for organisations that cannot send their data to a cloud provider: a self-hosted LLM (Ollama), connected to real systems through MCP servers, able to act and not only answer, with the guardrails of someone who ran critical infrastructure (least privilege, audit logs, human confirmation before any sensitive action). Everything below is public code you can read and run.

# Mohamed-Amine Rouabah (amineutron)

**Ingénieur IA locale et automatisation, Neutron Core**
Des assistants IA qui tournent chez vous, branchés sur vos outils, avec des garde-fous vérifiables.

[![Lyra CI](https://github.com/amineutron/lyra/actions/workflows/tests.yml/badge.svg)](https://github.com/amineutron/lyra/actions/workflows/tests.yml) [![Lyra: AGPL-3.0](https://img.shields.io/badge/Lyra-AGPL--3.0-blue.svg)](https://github.com/amineutron/lyra/blob/main/LICENSE) [![MCP servers: MIT](https://img.shields.io/badge/MCP%20servers-MIT-blue.svg)](https://github.com/amineutron?tab=repositories)

## Le problème, la preuve, la suite

Beaucoup d'organisations veulent un assistant IA mais ne peuvent pas envoyer leurs données chez un fournisseur cloud : secret professionnel, santé, propriété industrielle, exigences clients.
J'ai construit [Lyra](https://github.com/amineutron/lyra), un assistant DevOps vocal qui tourne entièrement sur une machine locale et pilote des machines virtuelles, des sauvegardes et des équipements, avec confirmation humaine avant chaque action sensible : le code, les tests et l'installeur sont publics.
Je propose la même architecture, adaptée à vos outils, en mission courte ou en régie (voir en bas de page).

## Une mesure plutôt qu'une promesse

Le modèle qui choisit l'outil dans Lyra fait 0,5 milliard de paramètres (800 Mo, 2 à 5 s par requête). Au départ il ne trouvait le bon outil pour aucune des 21 commandes que les règles ne couvraient pas. Vingt-et-une itérations d'une boucle mesurée (hypothèse écrite avant, mesure du mécanisme sans le modèle, jeux de test scellés avant de toucher au code) l'ont porté à 21/21, puis 100 % sur cinq jeux de développement ; sur un jeu scellé jamais itéré, il fait **42/50**. Le message n'est pas « 100 % » : c'est **70 à 85 % sur du langage jamais vu, avec un modèle de 800 Mo, en corrigeant ce qu'on lui montre et pas le modèle**.

![Meilleure configuration par itération](https://raw.githubusercontent.com/amineutron/lyra/main/docs/articles/figures/progression.svg)

Méthode, chiffres et limites : [l'article](https://github.com/amineutron/lyra/blob/main/docs/articles/2026-09-boucle-ephaistos.fr.md) ([English](https://github.com/amineutron/lyra/blob/main/docs/articles/2026-09-boucle-ephaistos.en.md)), tous les résultats bruts dans [`benchmarks/results/`](https://github.com/amineutron/lyra/tree/main/benchmarks/results).

## Ce que le code montre

| Dépôt | Rôle | État honnête |
|---|---|---|
| [lyra](https://github.com/amineutron/lyra) | Assistant DevOps vocal local, *French-first* : le français familier est compris par des règles avant d'appeler un modèle de 0,5B. Ollama, faster-whisper, Piper, RAG à 3 niveaux, démon multi-clients, installeur multi-distro | v1.3.0, AGPL-3.0, [![Tests](https://github.com/amineutron/lyra/actions/workflows/tests.yml/badge.svg)](https://github.com/amineutron/lyra/actions/workflows/tests.yml) |
| [fedora-agents](https://github.com/amineutron/fedora-agents) | Serveur MCP pour KVM/libvirt et sauvegardes Borg/Timeshift (TypeScript), table de permissions par outil | v1.2.0, MIT, composant de Lyra [![CI](https://github.com/amineutron/fedora-agents/actions/workflows/ci.yml/badge.svg)](https://github.com/amineutron/fedora-agents/actions/workflows/ci.yml) |
| [mcp-tracking](https://github.com/amineutron/mcp-tracking) | Serveur MCP, API HTTP et tableau de bord terminal pour suivre les tâches longues | v0.1.0, MIT [![CI](https://github.com/amineutron/mcp-tracking/actions/workflows/tests.yml/badge.svg)](https://github.com/amineutron/mcp-tracking/actions/workflows/tests.yml) |
| [neutroncore](https://github.com/amineutron/neutroncore) | Hub PWA du homelab (services, tâches, médias, domotique), piloté par Lyra | v0.1.0, MIT, son backend n'est pas encore publié [![CI](https://github.com/amineutron/neutroncore/actions/workflows/ci.yml/badge.svg)](https://github.com/amineutron/neutroncore/actions/workflows/ci.yml) |
| [hue-mcp](https://github.com/amineutron/hue-mcp) | Fork de [ThomasRohde/hue-mcp](https://github.com/ThomasRohde/hue-mcp) avec scènes par nom et synchronisation au rythme | MIT, fork crédité [![CI](https://github.com/amineutron/hue-mcp/actions/workflows/tests.yml/badge.svg)](https://github.com/amineutron/hue-mcp/actions/workflows/tests.yml) |
| [pylips-mcp](https://github.com/amineutron/pylips-mcp) | Serveur MCP pour TV Philips (JointSpace, ADB), certificat de la TV épinglé | v0.1.0, MIT [![CI](https://github.com/amineutron/pylips-mcp/actions/workflows/tests.yml/badge.svg)](https://github.com/amineutron/pylips-mcp/actions/workflows/tests.yml) |
| [denon-mcp](https://github.com/amineutron/denon-mcp) | Serveur MCP pour ampli home cinéma Denon (protocole telnet local) | v0.1.0, MIT [![CI](https://github.com/amineutron/denon-mcp/actions/workflows/tests.yml/badge.svg)](https://github.com/amineutron/denon-mcp/actions/workflows/tests.yml) |
| [catt-mcp](https://github.com/amineutron/catt-mcp) | Serveur MCP pour diffuser sur Chromecast et DLNA | v0.1.0, MIT [![CI](https://github.com/amineutron/catt-mcp/actions/workflows/tests.yml/badge.svg)](https://github.com/amineutron/catt-mcp/actions/workflows/tests.yml) |

Les serveurs domotiques sont taillés pour mon homelab ; ils servent de banc d'essai à l'architecture, pas de produit.

## Les garde-fous, dans le code

- Liste unique des outils dangereux, jamais auto-confirmés, même en mode performance : [`lyra/core/constants.py`](https://github.com/amineutron/lyra/blob/main/lyra/core/constants.py)
- Validation par liste blanche de tout argument transmis à un script shell : [`lyra/core/validation.py`](https://github.com/amineutron/lyra/blob/main/lyra/core/validation.py)
- Scripts privilégiés copiés en root et autorisés un par un dans `sudoers.d` : [`installer/core/steps/mcps.py`](https://github.com/amineutron/lyra/blob/main/installer/core/steps/mcps.py)
- Chaque outil MCP de fedora-agents déclare s'il est dangereux et s'il exige sudo, et le serveur refuse de démarrer si un outil n'est pas déclaré : [`src/config.ts`](https://github.com/amineutron/fedora-agents/blob/main/src/config.ts)

## Comment vérifier, en trois commandes

```bash
git clone https://github.com/amineutron/lyra && cd lyra && uv sync --extra dev && uv run pytest tests/unit -q   # les tests
grep -rn "https\?://" lyra modules | grep -v "127.0.0.1\|localhost"                                          # aucune sortie cloud
cat docs/DATA_FLOWS.md                                                                                       # ce qui est capté, gardé, et ce qui sort
```

Les garde-fous sont décrits dans le README de Lyra avec, pour chaque garantie, le fichier et le test qui la prouvent.

## Stack

- Agents et LLM local : Python, Ollama (Qwen 2.5 Coder, Llama 3.2), FastMCP et MCP, ChromaDB, BM25
- Voix : faster-whisper, Piper
- Infrastructure : Linux (Fedora, Debian), KVM et libvirt, Borg, systemd, n8n
- Interfaces : TypeScript, React, PWA, Textual
- Sécurité : droits minimaux, journalisation, confirmation humaine, durcissement Linux

## D'où je viens

Ingénieur diplômé de l'ESGI en alternance. Deux ans chez Canal+ Telecom sur une infrastructure critique de plus de 1 300 serveurs. Aujourd'hui indépendant sous la marque Neutron Core, en Île-de-France, à distance ou sur site, en français et en anglais.

## Ce que je propose

- **Diagnostic IA souveraine** : cas d'usage priorisés et preuve de concept sur vos données, sur votre matériel.
- **Agent IA sur mesure** : un assistant local connecté à vos outils (ERP, ticketing, supervision, infra), qui agit avec confirmation, traçabilité et droits maîtrisés.
- **Automatisation et DevOps** : Linux, KVM, sauvegardes, n8n, Python ; moins d'opérations manuelles, une infra documentée et surveillée.

## Contact

[Écrivez-moi](mailto:amine.neutroncore@gmail.com?subject=Contact%20via%20GitHub) ou retrouvez-moi sur [LinkedIn](https://www.linkedin.com/in/marouabah). Pour une faille de sécurité, suivez la [politique de sécurité](https://github.com/amineutron/.github/blob/main/SECURITY.md).
