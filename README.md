# mdefenderoffice
cript PowerShell para Adicionar Domínios na Tenant Allow/Block List (Microsoft 365 Defender)

#  Script PowerShell para Adicionar Domínios na Tenant Allow/Block List (Microsoft 365 Defender)

Este script PowerShell permite adicionar domínios (ou endereços de e-mail) à **Tenant Allow/Block List** do Microsoft Defender para Office 365, ignorando vereditos de spam, phishing e mensagens em massa com base em inteligência artificial.  

Ideal para times de segurança e e-mail que lidam com falsos positivos no filtro de proteção.

---

##Requisitos

- Permissão no tenant para executar `New-TenantAllowBlockListItems`.
- Módulo `ExchangeOnlineManagement` instalado.
- Autenticação via `Connect-ExchangeOnline`.

---

## O que o script faz?

- Lê domínios de um arquivo `.txt` separado por vírgulas.
- Remove espaços extras e entradas vazias.
- Divide os domínios em blocos de até 20 (limite do Microsoft Defender).
- Adiciona os domínios à **lista de permissões** (`Allow`) com expiração automática de 45 dias após o último uso.

---

## 🗂 Exemplo de arquivo `dominios.txt`

Crie um arquivo `.txt` com os domínios separados por vírgulas, **em uma única linha**, como no exemplo abaixo:


Salve o arquivo como `dominios.txt` em um local acessível, como `C:\Scripts\dominios.txt`.

---
## 🛠️ Script PowerShell
# Conectar ao Exchange Online
Connect-ExchangeOnline -UserPrincipalName <seu_user_principal_name>

# Caminho do arquivo com os domínios
$domainsRaw = Get-Content -Path "C:\Scripts\dominios.txt"

# Separar, limpar e preparar os domínios
$domains = $domainsRaw -join "," -split "," | ForEach-Object { $_.Trim() } | Where-Object { $_ -ne "" }

# Dividir em grupos de até 20 domínios (limite do Defender)
$domainGroups = [System.Collections.Generic.List[Object]]::new()
for ($i = 0; $i -lt $domains.Count; $i += 20) {
    $group = $domains[$i..([Math]::Min($i + 19, $domains.Count - 1))]
    $domainGroups.Add($group)
}

# Inserir cada grupo na Allow List com expiração de 45 dias
foreach ($group in $domainGroups) {
    Write-Host "Adicionando grupo com domínios: $($group -join ', ')" -ForegroundColor Cyan
    New-TenantAllowBlockListItems -ListType Sender -Allow -Entries $group -RemoveAfterDays 45
#}

Resultado Esperado
Cada grupo de até 20 domínios será adicionado à lista de permissões, e permanecerá ativo por 45 dias após o último uso.

A alteração será visível no portal do Microsoft Defender em:
https://security.microsoft.com/securitysettings/emailandcollab/emailallowblocklist

 Dica Extra
Se quiser validar rapidamente se um domínio já está na list
Get-TenantAllowBlockListItems -ListType Sender -ListSubType Allow | Where-Object { $_.Entries -like "*dominio.com*" }

Créditos
Criado por [Seu Nome], com base em necessidades reais de automação para segurança de e-mails no Microsoft 365.
Contribuições são bem-vindas!


