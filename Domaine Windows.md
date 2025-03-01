# Authentification
## NTLM relay
https://support.microsoft.com/en-us/topic/kb5005413-mitigating-ntlm-relay-attacks-on-active-directory-certificate-services-ad-cs-3612b773-4043-4aa9-b23d-b87910cd3429
Blocage des authentifications NTLM sortantes des systèmes privilégiés (contrôleur de domaine).
Audit des auth. existantes puis blocage.
### Journalisation des auth. NTLM
#### Génération des événements d'auth.
Configuration ordinateur > Stratégies > Paramètres Windows > Paramètres de sécurité > Stratégies locales > Options de sécurité > Sécurité réseau : Restreindre NTLM : Trafic NTLM sortant vers des serveurs distants ->Définir ce paramètre de stratégie : Auditer tout
**=>événement** : Microsoft-Windows-NTLM/8001 niv. Information
#### Lecture par eventvwr.msc
Journaux des applications et services > Microsoft > Windows > NTLM
#### Lecture du journal en Powershell
`Get-WinEvent -FilterHashTable @{LogName= »Microsoft-Windows-NTLM/Operational »;ID=8001} | Select MachineName,@{Name= »Target »; Expression={$_.Properties[0].Value}}`
### Blocage des auth. sortantes
#### Création d'une GPO sur les postes ciblés
Configuration ordinateur > Stratégies > Paramètres Windows > Paramètres de sécurité > Stratégies locales > Options de sécurité > Sécurité réseau : Restreindre NTLM : Trafic NTLM sortant vers des serveurs distants > Définir ce paramètre de stratégie : Refuser tout
**=>événement** : Microsoft-Windows-NTLM/4001 niv Avertissement
## Kerberos 
## LAPS
**Local Administrator Password Solution**
Gestion des mots de passes de comptes à privilèges, évite le rejeux dans le domaine.
On définit une stratégie de mot de passe pour éviter le rejeux de mots de passe administrateurs locaux.
### Installation sur le DC.
On ajoute l'utilisateur **admin** au groupe administrateur de schéma.
Récupération du contrôleur de domaine qui dispose du rôle FSMO ,import du module et mise à jour du schéma AD.
`Get-ADForest | Select-Object Name ShemaMaster`
### Attribution des droits d'écriture
On attribue les droits d'écriture aux machines pour leur permettre de mettre à jour les attributs.
`Set-AdmPwdComputerSelfPermission -OrgInit "CN=computer, DC=test, DC=test3"`
### Création de la GPO LAPS
Déplacement des deux fichiers de configuration
ADMX/ADML.
```
copy C:\Windows\PolicyDefinitions\AdmPwd.admx C:\Windows\SYSVOL\sysvol\it-connect.local\Policies\PolicyDefinitions\
copy C:\Windows\PolicyDefinitions\en-US\AdmPwd.adml C:\Windows\SYSVOL\sysvol\it-connect.local\Policies\PolicyDefinitions\en-US\
```

*Configuration ordinateur > Stratégies > Modèles d'administration > LAPS*
On applique cette GPO sur des machines et non des utilisateurs.
Modification de la GPO pour ajouter le fichier d'installation client.

- Création du partage *Package* et ajout des droits en lectures à tout utilisateurs identifiés
- Installation sur le client
Sur la machine cliente on se connecte en administrateur puis on force la mise à jour de GPO depuis le cmd.
``gpupdate /force``
Sur le DC on force la réinitialisation du mot de passe en Powershell.
`Reset-AdmPwdPassword -ComputerName PC1
On redémarre la station PC31 et on vérifie l'attribution du nouveau mot de passe.
`Get-AdmPwdPassword PC1`
Les mot de passe Administrateur locaux, ayant été utilisés pour la première attaque sont modifiés par LAPS appliqué par GPO. L'attaque initiale n'est plus faisable car les identifiants ont changés.
## CBT
**Channel Binding Token**
https://www.it-connect.fr/cours/formation-gratuite-microsoft-laps-installation-configuration/
# Signature des flux
## gMSA
**Group Managed Service Account**
Gestion des mots de passe par l'AD, 120 caractères et rotation automatique.
## Création d'un compte **gMSA**
On créer le groupe gMSA avec les paramètres suivants :
- DNSHostName : serveur sur lequel le groupe va être utilisé (srv11)
- ManagedPasswordIntervalInDays : durée pour le renouvellement du mot de passe

``New-ADServiceAccount -Name "gMSA-INSA" -Description "gMSA pour la tâche planifiée MS-CYNU INSA" -DNSHostName "srv11.avengers.assemble" -ManagedPasswordIntervalInDays 30 -PrincipalsAllowedToRetrieveManagedPassword "srv11$" -Enabled $True``
Vérification de la création du groupe dans l'AD : *Utilisateurs et ordinateurs AD > avengers.assemble > Managed Service Account*

Ajout de l'équipement **srv11** au groupe crée pour qu'il accède aux propriétés qui vont être donnés par la suite au groupe.
``Add-ADComputerServiceAccount -Identity srv11 -ServiceAccount gMSA-INSA``

Vérification : *Utilisateurs et ordinateurs AD > avengers.assemble > Computers > SRV11 > Editeur d'attibuts*
## Installation du GSMA
```
Add-WindowsFeature RSAT-AD-PowerShell
Install-ADServiceAccount gMSA-INSA
Test-AdServiceAccount -Identity gMSA-INSA
```
La commande nous retourne **true**, cela indique sa bonne installation.
https://akril.net/utilisation-des-comptes-gmsa-dans-lactive-directory/
# Partage
## Restriction d'accès aux partages
Une autre faille est l'accès à un partage réseau qui contient des données sensibles. Une solution est de restreindre son accès. Ce partage est à l'origine pour héberger une ressource utilisée par une tache planifiée.
Pour cela, on utilise un compte de service administré de groupe (gMSA) pour accéder au partage Script.
## Génération d'une clé de groupe
On créer une clé racine KDS qui va être utilisée par le groupe pour s'authentifier (mécanisme Kerberos).
``Add-KdsRootKey -EffectiveImmediately ((Get-Date).AddHours(-10))``
On vérifie que la clé a bien été générée.
*Sites et services AD > Services > Group Key Distribution Service > Master Root Keys*
# BitLocker
Sur le DC on installe la fonctionnalité **Bitlocker** sur le domaine.
``Install-WindowsFeature -Name "RSAT-Feature-Tools-BitLocker", "RSAT-Feature-Tools-BitLocker-RemoteAdminTool", "RSAT-Feature-Tools-BitLocker-BdeAductExt"
![[tp3_bitlockerFeature.jpg]]
On créer ensuite la GPO pour appliquer Bitlocker sur un groupe définit.
![[tp3_bitlocker.jpg]]

On paramètre la GPO avec les différentes fonctionnalités BitLocker à appliquer.
*Configuration ordinateur > stratégies > modèles d'administration > compasants Windows > Chiffrement de lecteurs Bitlocker*
![[tp3_bitlockerConfig.jpg]]

On choisi le stockage des clés de récupération BitLocker dans l'Active Directory.
On active manuellement le chiffrement sur le poste **srv30**.
# Résolution de nom
## LLMNR
https://techexpert.tips/fr/windows-fr/gpo-desactiver-llmnr-sous-windows/