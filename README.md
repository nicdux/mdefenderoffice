#  Script PowerShell para Adicionar Domínios na Tenant Allow/Block List (Microsoft 365 Defender)

Este script PowerShell permite adicionar domínios (ou endereços de e-mail) à **Tenant Allow/Block List** do Microsoft Defender para Office 365, ignorando vereditos de spam, phishing e mensagens em massa com base em inteligência artificial.  

Ideal para times de segurança e e-mail que lidam com falsos positivos no filtro de proteção.

---

# Criar script PowerShell comentado
powershell_script = """# ===============================================
# Script: Add-TenantAllowBlockList.ps1
# Autor: [Seu Nome]
# Descrição:
#     Este script adiciona domínios ou endereços de e-mail
#     à lista de permissões (Allow) da Tenant Allow/Block List
#     do Microsoft Defender para Office 365.
#     Ele aceita um arquivo .txt com os domínios separados por vírgula
#     e os adiciona em blocos de até 20 entradas, como exigido pela plataforma.
# ===============================================

# -------- CONFIGURAÇÃO --------

# Conectar ao Exchange Online com uma conta com permissões adequadas
Connect-ExchangeOnline -UserPrincipalName <seu_user_principal_name>

# Defina o caminho para o arquivo que contém os domínios separados por vírgula
$domainsRaw = Get-Content -Path "C:\\Scripts\\dominios.txt"

# -------- PROCESSAMENTO DOS DOMÍNIOS --------

# Junta todos os dados em uma única string (caso haja múltiplas linhas),
# separa por vírgula, remove espaços e entradas vazias
$domains = $domainsRaw -join "," -split "," | ForEach-Object { $_.Trim() } | Where-Object { $_ -ne "" }

# Cria uma lista de grupos com no máximo 20 domínios cada
$domainGroups = [System.Collections.Generic.List[Object]]::new()
for ($i = 0; $i -lt $domains.Count; $i += 20) {
    $group = $domains[$i..([Math]::Min($i + 19, $domains.Count - 1))]
    $domainGroups.Add($group)
}

# -------- INSERÇÃO NA ALLOW LIST --------

# Para cada grupo de até 20 domínios, executa o cmdlet de inserção
foreach ($group in $domainGroups) {
    Write-Host "Adicionando grupo com domínios: $($group -join ', ')" -ForegroundColor Cyan
    
    # Adiciona os domínios com expiração automática de 45 dias após o último uso
    New-TenantAllowBlockListItems -ListType Sender -Allow -Entries $group -RemoveAfterDays 45
}

# -------- FIM --------
Write-Host "Processo concluído com sucesso." -ForegroundColor Green
"""

# Salvar o script em um arquivo .ps1
powershell_path = Path("/mnt/data/Add-TenantAllowBlockList.ps1")
powershell_path.write_text(powershell_script, encoding="utf-8")

# Também salvar o exemplo de domínio
example_domains = "pacificaint.com,alerte.com.br,gympass.com,gympass.commail-seguro.com,mail-sec.com,securityapp.cloud,ultra.2fa.com-token-auth.com,ultra.mail.kb4.io,ultra.com.br"
example_path = Path("/mnt/data/dominios.txt")
example_path.write_text(example_domains, encoding="utf-8")

# Compactar os arquivos para ZIP
import zipfile

zip_path = "/mnt/data/tenant-allowlist-script.zip"
with zipfile.ZipFile(zip_path, "w") as zipf:
    zipf.write(powershell_path, arcname="Add-TenantAllowBlockList.ps1")
    zipf.write(example_path, arcname="dominios.txt")

zip_path
