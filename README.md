from pathlib import Path

# Conteúdo do Markdown
markdown_content = """# 🚀 Script PowerShell para Adicionar Domínios na Tenant Allow/Block List (Microsoft 365 Defender)

Este script PowerShell permite adicionar domínios (ou endereços de e-mail) à **Tenant Allow/Block List** do Microsoft Defender para Office 365, ignorando vereditos de spam, phishing e mensagens em massa com base em inteligência artificial.  

✅ Ideal para times de segurança e e-mail que lidam com falsos positivos no filtro de proteção.

---

## 📌 Requisitos

- Permissão no tenant para executar `New-TenantAllowBlockListItems`.
- Módulo `ExchangeOnlineManagement` instalado.
- Autenticação via `Connect-ExchangeOnline`.

---

## 🧠 O que o script faz?

- Lê domínios de um arquivo `.txt` separado por vírgulas.
- Remove espaços extras e entradas vazias.
- Divide os domínios em blocos de até 20 (limite do Microsoft Defender).
- Adiciona os domínios à **lista de permissões** (`Allow`) com expiração automática de 45 dias após o último uso.

---

## 🗂️ Exemplo de arquivo `dominios.txt`

Crie um arquivo `.txt` com os domínios separados por vírgulas, **em uma única linha**, como no exemplo abaixo:


zip_path
