Dans des parcs Microsofts, trois protocoles de résolutions de noms peuvent prendres le relais du DNS si celui-ci n'est pas en capacité à trouver un ressource.

Outil Powershell pour trouver un faux serveur DNS, à lancer régulièrement (tâche planifiée).

````
$llmnr = (Resolve-DnsName -LlmnrOnly falseUserRequest 2> $Null)
if ($llmnr) {
$ip = $llmnr.IpAddress -Join ", "
$msg="Hostname: ${env:computername} `nRogue LLMNR Server: $ip"
$body=ConvertTo-Json @{
  username="Responder"
  pretext="Detection d'un serveur DNS illégitime"
  text=$msg
}
Invoke-RestMethod https://monSrv.local/ -Method Post -Body $body -ContentType 'application/json'
}
````

Resolve-DnsName -LlmnrOnly falseUserRequest //Envoi d'une fausse requête DNS
