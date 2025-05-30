Documentation du Projet PXE - Déploiement Automatisé de Workstations Windows

📌 Introduction
Ce projet a pour objectif de mettre en place une infrastructure permettant le déploiement automatique de stations de travail Windows via un serveur PXE (Preboot eXecution Environment). L'ensemble comprend :

Un serveur PXE avec WDS (Windows Deployment Services)

Un domaine Active Directory

Des images de déploiement personnalisées

Des scripts PowerShell d'automatisation

📚 Job 0 - Recherches Préliminaires
PowerShell Basics
Extensions de fichiers :

.ps1 : Script PowerShell standard

.psm1 : Module PowerShell

.ps1xml : Fichier XML pour formatage ou types étendus

.psd1 : Fichier de données/manifeste de module

.pssc : Configuration de session PowerShell

Comparaison avec Bash :

PowerShell utilise des cmdlets plutôt que des commandes UNIX

Pipeline transporte des objets plutôt que du texte

Fonctionne sous Linux via PowerShell Core

Infrastructure PXE
Composants nécessaires :

Serveur DHCP (peut être intégré à WDS)

Serveur TFTP (généralement inclus dans WDS)

Stockage pour les images

Windows ADK pour la création d'images

Windows ADK : Kit d'évaluation et de déploiement Windows, contient les outils pour créer et personnaliser des images Windows.

WDS : Windows Deployment Services, service Microsoft pour le déploiement réseau de systèmes Windows.

🛠️ Job 1 - Configuration du Serveur
Prérequis
Windows Server 2016/2019/2022

4+ vCPU, 8+ GB RAM, 100+ GB stockage

2 cartes réseau (recommandé)

Étapes d'installation
Installation des rôles :

powershell
Install-WindowsFeature -Name AD-Domain-Services,DHCP,DNS,WDS -IncludeManagementTools
Configuration AD :

powershell
Install-ADDSForest -DomainName "votredomaine.local" -InstallDNS
Installation Windows ADK :

Télécharger depuis Microsoft

Configuration WDS :

powershell
Initialize-WDS -Server "localhost" -Path "D:\RemoteInstall"
Add-WDSServer -Directory "D:\RemoteInstall"
Tests de validation
powershell
Test-NetConnection -ComputerName localhost -Port 69  # Test TFTP
Get-DhcpServerv4Scope  # Vérification scope DHCP
Get-WdsInstallImage   # Liste des images disponibles

🖼️ Job 2 - Création d'Images
Création d'image WinPE
Générer l'environnement :

powershell
copype amd64 D:\WinPE_amd64
Personnaliser avec ADK :

powershell
Add-WindowsPackage -Path "D:\WinPE_amd64\mount" -PackagePath "C:\Program Files (x86)\Windows Kits\10\Assessment and Deployment Kit\Windows Preinstallation Environment\amd64\WinPE_OCs\WinPE-WMI.cab"
Exporter en image WIM :

powershell
dism /export-image /sourceimagefile:D:\WinPE_amd64\media\sources\boot.wim /sourceindex:1 /destinationimagefile:D:\PXE_Images\WinPE_Custom.wim
Ajout au serveur WDS
powershell
Add-WdsInstallImage -Path "D:\PXE_Images\WinPE_Custom.wim" -ImageName "WinPE Custom" -ImageGroup "PXE_Images"
💻 Job 3 - Déploiement Client
Processus de boot PXE
Client envoie requête DHCP

Serveur répond avec adresse IP et info PXE

Téléchargement du bootloader via TFTP

Chargement de l'image sélectionnée

Intégration Automatique au Domaine
Script PowerShell (Join-Domain.ps1) :

powershell
$domain = "votredomaine.local"
$cred = New-Object System.Management.Automation.PSCredential("admin", (ConvertTo-SecureString "MotDePasse" -AsPlainText -Force))
Add-Computer -DomainName $domain -Credential $cred -Restart
Sécurité PXE
Mesures implémentées :

Signature des images avec Authenticode

Filtrage MAC address

WDS : Approbation administrative requise

Journalisation complète des déploiements

powershell
# Exemple de configuration sécurité
Set-WdsClient -Approval Administrator
Set-WdsBootImage -SecureBoot Enabled

Ce projet nous a permis de mettre en place une infrastructure complète de déploiement automatisé de stations de travail Windows via un serveur PXE. Grâce à l’utilisation de Windows Deployment Services (WDS), d’Active Directory et de scripts PowerShell, nous avons automatisé l’installation et l’intégration des machines dans un domaine, réduisant ainsi les interventions manuelles et accélérant le déploiement à grande échelle.

Bilan des Acquis
✔ Maîtrise de l’écosystème PXE : Compréhension des protocoles DHCP, TFTP et du processus de boot réseau.
✔ Automatisation PowerShell : Création de scripts pour la configuration du serveur et l’intégration des clients au domaine.
✔ Gestion d’images : Utilisation de Windows ADK pour générer et personnaliser des images WinPE et Windows.
✔ Sécurisation du déploiement : Mise en place de restrictions d’accès et de vérification d’intégrité des images.

Perspectives d’Amélioration
🔹 Intégration avec MDT (Microsoft Deployment Toolkit) pour une gestion encore plus avancée des déploiements.
🔹 Déploiement hybride (cloud/on-premise) avec Azure DevOps pour des scénarios hybrides.
🔹 Automatisation renforcée via Ansible ou DSC (Desired State Configuration) pour une gestion infrastructure-as-code.

Ce projet illustre parfaitement l’importance de l’automatisation dans l’administration système, permettant de gagner en efficacité tout en garantissant une meilleure cohérence des environnements déployés.

