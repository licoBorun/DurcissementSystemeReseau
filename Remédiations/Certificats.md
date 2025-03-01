# Définition
Ensemble de 
- composants physiques (ordinateurs, HSM, cartes à puces)
- humain (certifications)
- logiciel (système, application)
pour identifier une chose
La base est la confiance du certificat (et de la chaîne).
Les différents champs
- identification du certificat
	- clé publique
	- num. de série
	- émetteur
	- nom du sujet
	- id. de la clé de l'autorité et du sujet 
- contrôle de validité
	- date début/fin validité
	- pt de distribution de la CRL (CRLDP) : URL listant une liste de certificats invalides
	- Info d'accès à l'autorité (AIA)
		- Attribut AC émettrice : URL indiquant le certificat du signataire
		- Attribut OSCP (Online Status Certificate Protocol): URL du service de validation idem que CRL mais instantané
- restriction d'usage
	- contraintes de base
	- usage de la clé
	- u. étendu
	- criticité des champs de restriction d'usage
- informations supplémentaires
	- stratégie de certificat
- champs spécifique
	- autre nom de l'objet
	- OID propriétaire

FIDO2

|   |   |
|---|---|
|CA (Certificat Authority)|Autorité de certification|
|KEY (Private key)|Clé privée|
|CSR (Certificat Signing Request)|Demande de signature|
|CRT|Certificat|

Format de certificat X.509
PKCS (Public-Key Cryptography Standards) format des clés
Encodage PEM et DER (binaire)
PKI
CLM (Certificate Lifecycle Management)