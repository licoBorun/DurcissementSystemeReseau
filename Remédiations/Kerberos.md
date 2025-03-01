# Récupération de tickets
Le point faible exploité par ces attaques est la robustesse du mot de passe. Une fois qu'une personne malveillante a accès à un ticket Kerberos il va tenter de casser le condensat par une attaque de brute force dans l'objectif de trouver le mot de passe en clair. Cette attaque est d'autant plus efficace que le mot de passe est faible.
## AS-RepRoasting (quick-win)
Afin de pallier ce défaut de configuration, il est nécessaire de changer le paramètre de la pré-authentification pour chaque utilisateur.
### Lister les utilisateurs concernés
Avant d'appliquer une règle désactivant la pré-authentification sur l'ensemble des utilisateurs du domaine, il est recommandé d'établir une liste des comptes concernés.

Une commande PowerShell à lancer sur le contrôleur de domaine permet de lister les comptes dont la pré-authentification n'est pas requise : 
`Get-ADUSer -Filter 'DoesNotRequirePreAuth -eq $true'`

Ci-dessous un résultat sur un domaine de test.
![](as-reproasting_list_preAuth.png)
A partir de cette liste, vous pouvez juger si la désactivation de la pré-authentification est nécessaire on non pour chacun des comptes.
### Modification du paramètre pour chaque utilisateur
Pour chaque utilisateur concerné on applique le paramètre d'activation de la pré-authentification.
`Utilisateur et ordinateurs Active Directory > Propriété du compte puis l'onglet Compte`
Il faut ensuite décocher la dernière case : `La pré-authentification Kerberos n'est pas nécessaire`
![](pre_auth_activation.png)
## Kerberoasting
### Liste des utilisateurs possédants un SPN
L'exécution du script PowerShell ci-dessous aura pour effet de lister l'ensemble des utilisateurs du domaine possédant un ou plusieurs SPN.
```
$search = New-Object DirectoryServices.DirectorySearcher([ADSI]"")
$search.filter = "(&(objectCategory=person)(objectClass=user)(servicePrincipalName=*))"
$results = $search.Findall()
foreach($result in $results)
{
	$userEntry = $result.GetDirectoryEntry()
	Write-host "User : " $userEntry.name "(" $userEntry.distinguishedName ")"
	Write-host "SPNs"        
	foreach($SPN in $userEntry.servicePrincipalName)
	{
		$SPN       
	}
	Write-host ""
}
```

Ci-dessous un résultat sur un domaine de test.
![](kerberoasting_list_SPN.png)
### Suppression des SPN liés à des utilisateurs
Pour afficher les SPN associés à des comptes utilisateurs il faut afficher les fonctionnalités avancées.
![](aff_fonctionAdv.png)
On peut ensuite venir dans l'onglet `Editeur d'attributs` pour chaque utilisateur concerné et ainsi supprimer les SPN associés.
![](kerberoasting_deleteSPN.png)
## Délégation
