# Checklist Conformité et Sécurité

## Vue d'Ensemble

Ce document présente les checklists complètes pour garantir la conformité légale et la sécurité de votre SAS proposant des services d'IA.

## 1. Checklist Légale - Constitution SAS

### Avant Immatriculation

- [ ] **Choix de la dénomination sociale**
  - Vérifier disponibilité sur inpi.fr
  - Vérifier domaine web disponible (.fr ou .com)
  - Pas de confusion avec marques existantes

- [ ] **Rédaction des statuts**
  - Dénomination sociale
  - Forme juridique (SAS)
  - Siège social (adresse complète)
  - Objet social (services d'intelligence artificielle)
  - Capital social (montant minimum 1€, recommandé 1000-5000€)
  - Durée de vie (99 ans maximum)
  - Pouvoirs du président
  - Modalités de décision
  - Répartition des actions

- [ ] **Nomination du président**
  - Personne physique majeure
  - Déclaration de non-condamnation
  - Acceptation des fonctions
  - Signature des statuts

- [ ] **Apport du capital**
  - Ouverture compte bancaire professionnel
  - Dépôt des fonds (espèces ou chèque)
  - Obtention certificat de dépôt
  - Conservation certificat (pour immatriculation)

### Immatriculation

- [ ] **Publication annonce légale**
  - Rédaction conforme (modèle JAL)
  - Publication dans JAL du département du siège
  - Conservation de l'attestation de parution
  - Budget : 150-250€

- [ ] **Constitution du dossier**
  - Formulaire M0 (ou déclaration en ligne)
  - Statuts signés (2 exemplaires)
  - Attestation de parution JAL
  - Certificat de dépôt des fonds
  - Déclaration de non-condamnation président
  - Pièce d'identité président
  - Justificatif siège social

- [ ] **Dépôt au guichet unique**
  - Via formalites.entreprises.gouv.fr
  - Paiement frais (~40€)
  - Suivi du dossier
  - Réception K-bis (3-7 jours)

### Post-Immatriculation

- [ ] **Déblocage des fonds**
  - Présentation K-bis à la banque
  - Virement sur compte professionnel
  - Activation moyens de paiement

- [ ] **Déclarations administratives**
  - Demande SIRET (automatique)
  - Code APE (automatique - 6209Z probable)
  - Affiliation URSSAF
  - Inscription CPAM (si président rémunéré)

## 2. Checklist RGPD - Protection des Données

### Analyse Préliminaire

- [ ] **Cartographie des données**
  - Lister toutes les données collectées
  - Identifier les données personnelles
  - Identifier les données sensibles
  - Mapper les flux de données

- [ ] **Registre des traitements**
  - Créer un registre conforme
  - Documenter chaque traitement
  - Identifier finalités
  - Identifier base légale
  - Durée de conservation

### Base Légale des Traitements

- [ ] **Consentement**
  - Formulaire de consentement explicite
  - Cases à cocher non pré-cochées
  - Possibilité de retrait facile
  - Conservation preuve du consentement

- [ ] **Contrat**
  - Clause RGPD dans CGV/CGU
  - Mention traitement nécessaire au service

- [ ] **Intérêt légitime**
  - Justification documentée
  - Test de proportionnalité

### Mesures Techniques et Organisationnelles

- [ ] **Sécurité des données**
  - Chiffrement en transit (TLS 1.3)
  - Chiffrement au repos (AES-256)
  - Hachage mots de passe (bcrypt)
  - Accès restreints (RBAC)
  - Logs d'accès
  - Sauvegardes régulières

- [ ] **Minimisation des données**
  - Collecter uniquement le nécessaire
  - Anonymisation si possible
  - Pseudonymisation des données
  - Suppression automatique (après délai)

- [ ] **Durée de conservation**
  - Définir durées par type de données
  - Automatiser la suppression
  - Archives légales (3 ans comptabilité)

### Droits des Personnes

- [ ] **Information des utilisateurs**
  - Politique de confidentialité accessible
  - Mentions lors de la collecte
  - Information claire et compréhensible

- [ ] **Droit d'accès**
  - Formulaire de demande d'accès
  - Procédure de vérification identité
  - Délai de réponse : 1 mois maximum
  - Format export des données

- [ ] **Droit de rectification**
  - Interface de modification profil
  - Procédure de rectification

- [ ] **Droit à l'effacement**
  - Bouton "Supprimer mon compte"
  - Suppression effective des données
  - Conservation limitée aux obligations légales

- [ ] **Droit à la portabilité**
  - Export données format structuré (JSON/CSV)
  - Transmission à autre responsable si demandé

- [ ] **Droit d'opposition**
  - Opt-out marketing
  - Opposition traitement

### Sous-traitants et Transferts

- [ ] **Contrats sous-traitants (DPA)**
  - Data Processing Agreement avec chaque sous-traitant
  - Clauses RGPD obligatoires
  - Audit régulier sous-traitants

- [ ] **Transferts hors UE**
  - Identifier tous les transferts
  - Clauses contractuelles types (SCC)
  - Privacy Shield invalide (US)
  - Préférer hébergement UE

### Gouvernance

- [ ] **DPO (Data Protection Officer)**
  - Obligatoire si traitement à grande échelle
  - Nomination interne ou externe
  - Contact publié

- [ ] **Procédure de violation de données**
  - Process de détection
  - Notification CNIL sous 72h
  - Notification personnes concernées si risque élevé
  - Registre des violations

- [ ] **Analyse d'impact (DPIA)**
  - Obligatoire si risque élevé
  - Documenter l'analyse
  - Consulter CNIL si résiduel élevé

### Conformité Continue

- [ ] **Audit annuel RGPD**
  - Vérification conformité
  - Mise à jour registre
  - Tests procédures

- [ ] **Formation équipe**
  - Sensibilisation RGPD
  - Bonnes pratiques sécurité
  - Mise à jour régulière

## 3. Checklist Sécurité Informatique

### Sécurité Réseau

- [ ] **Firewall**
  - UFW/iptables configuré
  - Règles restrictives (deny all, allow specific)
  - Ports ouverts : 80, 443, 22 uniquement
  - Logging activé

- [ ] **Protection DDoS**
  - Cloudflare ou équivalent
  - Rate limiting configuré
  - WAF (Web Application Firewall)

- [ ] **VPN**
  - Accès admin uniquement via VPN
  - Authentification forte

### Sécurité Serveurs

- [ ] **Système d'exploitation**
  - Ubuntu LTS ou Debian stable
  - Mises à jour automatiques (unattended-upgrades)
  - Suppression services inutiles
  - Hardening (CIS Benchmarks)

- [ ] **SSH**
  - Port non-standard (pas 22)
  - Authentification par clés uniquement
  - Désactivation root login
  - Fail2ban installé et configuré

- [ ] **Docker**
  - Images officielles uniquement
  - Scan vulnérabilités (Trivy, Clair)
  - User non-root dans containers
  - Secrets management (Docker secrets)

### Sécurité Application

- [ ] **Authentification**
  - JWT tokens avec expiration
  - Refresh tokens sécurisés
  - 2FA recommandé (optionnel)
  - Limitation tentatives login

- [ ] **Autorisation**
  - RBAC (Role-Based Access Control)
  - Principe du moindre privilège
  - Vérification permissions chaque requête

- [ ] **Validation données**
  - Validation côté serveur (Pydantic)
  - Sanitisation entrées utilisateur
  - Requêtes préparées (SQL injection)
  - Limitation taille uploads

- [ ] **Headers sécurité**
  - HTTPS obligatoire (redirection)
  - HSTS activé
  - X-Frame-Options: DENY
  - X-Content-Type-Options: nosniff
  - CSP (Content Security Policy)

- [ ] **Sessions**
  - Cookies HttpOnly
  - Cookies Secure
  - SameSite=Strict
  - Durée de vie limitée

### Sécurité Base de Données

- [ ] **Accès**
  - Pas d'accès direct depuis internet
  - User dédié par application
  - Principe moindre privilège

- [ ] **Chiffrement**
  - Données sensibles chiffrées (AES-256)
  - Mots de passe hashés (bcrypt)
  - Certificats SSL pour connexions

- [ ] **Sauvegardes**
  - Automatiques quotidiennes
  - Chiffrées
  - Stockage hors site
  - Tests de restauration mensuels

### Sécurité APIs Externes

- [ ] **Gestion clés API**
  - Variables d'environnement uniquement
  - Jamais dans le code source
  - Rotation régulière (90 jours)
  - Révocation si exposition

- [ ] **Rate limiting**
  - Limitation par utilisateur
  - Limitation par IP
  - Quotas mensuels

- [ ] **Monitoring usage**
  - Alertes sur usage anormal
  - Logs d'utilisation
  - Budget alerts

### Monitoring et Alertes

- [ ] **Uptime monitoring**
  - UptimeRobot ou Pingdom
  - Vérification toutes les 5 minutes
  - Alertes email/SMS

- [ ] **Application monitoring**
  - Sentry pour erreurs
  - Logs centralisés (ELK)
  - Métriques performance

- [ ] **Security monitoring**
  - OSSEC ou Wazuh (HIDS)
  - Alertes tentatives intrusion
  - Scan vulnérabilités hebdomadaire

### Gestion Incidents

- [ ] **Plan de réponse incident**
  - Procédure documentée
  - Rôles et responsabilités
  - Communication de crise

- [ ] **Backups et DR**
  - Plan de reprise d'activité (PRA)
  - RPO : 24h maximum
  - RTO : 4h maximum
  - Tests semestriels

## 4. Checklist Qualité Code

### Standards de Code

- [ ] **Python (Backend)**
  - PEP 8 respecté
  - Linting (flake8, pylint)
  - Formatting (black)
  - Type hints (mypy)

- [ ] **JavaScript/TypeScript (Frontend)**
  - ESLint configuré
  - Prettier pour formatting
  - TypeScript strict mode
  - Pas de console.log en production

### Tests

- [ ] **Tests unitaires**
  - Couverture > 80%
  - Tests rapides (< 1s)
  - Mock des dépendances externes

- [ ] **Tests d'intégration**
  - APIs testées
  - Base de données test
  - Scénarios complets

- [ ] **Tests de sécurité**
  - Bandit (Python)
  - npm audit (JavaScript)
  - OWASP ZAP (scan)

### Documentation

- [ ] **Code**
  - Docstrings (Google style)
  - Commentaires explicatifs si nécessaire
  - README à jour

- [ ] **API**
  - Documentation OpenAPI/Swagger
  - Exemples d'utilisation
  - Codes d'erreur documentés

- [ ] **Architecture**
  - Diagrammes à jour
  - Décisions techniques documentées
  - Procédures de déploiement

### CI/CD

- [ ] **Pipeline automatisé**
  - Tests automatiques
  - Linting automatique
  - Scan sécurité
  - Déploiement automatique

- [ ] **Environnements**
  - Dev / Staging / Production
  - Config par environnement
  - Données de test

## 5. Checklist Propriété Intellectuelle

### Code Source

- [ ] **Licence**
  - Choix de licence (MIT, Apache, Propriétaire)
  - Fichier LICENSE à la racine
  - Headers de copyright dans fichiers

- [ ] **Dépôt sécurisé**
  - Git repository privé
  - Contrôle d'accès strict
  - Backup du code

### Contenu IA

- [ ] **Attribution**
  - Disclaimer "Généré par IA"
  - Révision humaine obligatoire
  - Sources citées si applicable

- [ ] **Vérification plagiat**
  - Outil anti-plagiat (Copyscape)
  - Vérification manuelle
  - Originalité garantie

### Marques et Logos

- [ ] **Dépôt de marque**
  - Recherche d'antériorité
  - Dépôt INPI
  - Protection logo

## 6. Checklist Assurances

- [ ] **RC Professionnelle**
  - Contrat souscrit
  - Couverture 500k-1M€
  - Renouvellement automatique

- [ ] **Cyber-assurance**
  - Couvre violations données
  - Couvre ransomware
  - Assistance juridique incluse

- [ ] **Protection juridique**
  - Défense en cas de litige
  - Recouvrement créances

## 7. Checklist Fiscale

### Déclarations Obligatoires

- [ ] **TVA**
  - Inscription si CA > 36 500€
  - Déclaration mensuelle/trimestrielle
  - Paiement dans les délais

- [ ] **Impôt sur les sociétés**
  - Liasse fiscale annuelle
  - Télédéclaration
  - Acomptes trimestriels

- [ ] **CFE**
  - Déclaration initiale année N
  - Paiement année N+1

### Optimisations

- [ ] **CIR (Crédit Impôt Recherche)**
  - Dépenses R&D éligibles
  - Documentation technique
  - Déclaration annuelle

- [ ] **JEI (Jeune Entreprise Innovante)**
  - Vérification éligibilité
  - Demande de statut
  - Exonérations charges

## 8. Checklist Comptable

- [ ] **Tenue comptable**
  - Logiciel de comptabilité
  - Saisie régulière
  - Rapprochements bancaires mensuels

- [ ] **Justificatifs**
  - Factures clients numérotées
  - Factures fournisseurs classées
  - Relevés bancaires conservés

- [ ] **Comptes annuels**
  - Bilan
  - Compte de résultat
  - Annexes
  - Dépôt au greffe (6 mois après clôture)

## 9. Checklist Marketing et Commercial

- [ ] **Site web**
  - Mentions légales complètes
  - Politique de confidentialité
  - CGV/CGU
  - Cookies (bandeau conforme)

- [ ] **Contrats clients**
  - Modèle de contrat validé
  - Clause RGPD
  - Clause responsabilité
  - Conditions de paiement

- [ ] **Facturation**
  - Mentions obligatoires
  - Numérotation séquentielle
  - TVA si applicable
  - Conservation 10 ans

## Calendrier des Obligations

### Mensuelles
- Déclarations sociales (URSSAF)
- TVA (si régime mensuel)
- Rapprochement bancaire

### Trimestrielles
- TVA (si régime trimestriel)
- Acomptes IS

### Annuelles
- Liasse fiscale (mai N+1)
- Comptes annuels (dépôt greffe)
- Déclaration CIR
- Audit RGPD
- Renouvellement assurances

---

**Note** : Cette checklist est un guide. Consultez des professionnels (avocat, expert-comptable, DPO) pour votre situation spécifique.

**Version** : 1.0  
**Dernière mise à jour** : Février 2024
