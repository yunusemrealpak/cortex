---
title: Project Vault - SSH Sunucu Erişim Kurulumu
type: note
tags:
  - ssh
  - deployment
  - hetzner
  - windows
  - project-vault
  - devops
summary: >-
  Windows'tan Hetzner VPS'e (vault-server) SSH key ile şifresiz bağlantı
  kurulumu
relatedTerms: []
created: '2026-04-06T22:02:39.726Z'
updated: '2026-04-06T22:02:39.726Z'
---
# Project Vault - SSH Sunucu Erişim Kurulumu

> Windows'tan Hetzner VPS'e (vault-server) SSH key ile şifresiz bağlantı kurulumu

## İçerik


## Sunucu Bilgileri

- **IP:** 31.210.40.218
- **Kullanıcı:** deploy
- **SSH Alias:** vault-server
- **Konum:** Hetzner VPS, Ubuntu

## Yapılan Adımlar

### 1. SSH Key Oluşturma (Windows lokal)

```bash
ssh-keygen -t ed25519 -f ~/.ssh/id_server -C "deploy@vault-server"
```

- Passphrase boş bırakıldı
- Oluşan dosyalar: `~/.ssh/id_server` (private), `~/.ssh/id_server.pub` (public)

### 2. SSH Config'e Alias Ekleme

`C:\Users\byusn\.ssh\config` dosyasına eklenen blok:

```
Host vault-server
    HostName 31.210.40.218
    User deploy
    IdentityFile ~/.ssh/id_server
    IdentitiesOnly yes
```

Bu sayede `ssh vault-server` yazmak yeterli — IP, kullanıcı ve key otomatik kullanılıyor.

### 3. Public Key'i Sunucuya Ekleme

`deploy` kullanıcısının `.ssh` klasörü root'a ait olduğu için `deploy` ile yazma izni yoktu. **Root ile bağlanıp** düzeltildi:

```bash
ssh root@31.210.40.218 "chown -R deploy:deploy /home/deploy/.ssh && chmod 700 /home/deploy/.ssh && echo 'ssh-ed25519 AAAA...' >> /home/deploy/.ssh/authorized_keys && chmod 600 /home/deploy/.ssh/authorized_keys && chown deploy:deploy /home/deploy/.ssh/authorized_keys"
```

### 4. Karşılaşılan Sorunlar

| Sorun | Neden | Çözüm |
|-------|-------|-------|
| `Permission denied (publickey,password)` | Sunucuda SSH key tanımlı değildi | Key oluşturup authorized_keys'e eklendi |
| `Permission denied` on authorized_keys | `.ssh` klasörü root'a aitti, deploy yazamıyordu | Root ile `chown -R deploy:deploy /home/deploy/.ssh` |
| Windows'ta `ssh-copy-id` yok | Windows native SSH'ta bu komut bulunmuyor | Manuel `echo >> authorized_keys` ile eklendi |
| Script IP ile bağlanamıyor | SSH config'deki key sadece `vault-server` alias'ına tanımlıydı | Publish script'i alias kullanacak şekilde güncellendi |

### 5. Doğrulama

```bash
ssh vault-server "echo 'Bağlantı başarılı!'"
```

### 6. Mevcut SSH Key Yapısı

```
~/.ssh/
├── config              ← GitHub, GitLab, vault-server alias'ları
├── id_github / .pub    ← GitHub key
├── id_gitlab / .pub    ← GitLab key
├── id_server / .pub    ← Sunucu key (vault-server)
└── known_hosts
```

## Publish Script Entegrasyonu

SSH alias sayesinde deploy scripti artık şifresiz çalışıyor:

```bash
./scripts/publish.sh deploy   # vault-server alias'ını kullanır
```

Script içinde `REMOTE_HOST` varsayılan olarak `vault-server` ayarlı.


## Önemli Noktalar

- SSH key: ed25519 tipinde, ~/.ssh/id_server
- SSH alias: vault-server → 31.210.40.218 (deploy kullanıcısı)
- Windows'ta ssh-copy-id yok — manuel echo ile eklenir
- deploy kullanıcısının .ssh izinleri root ile düzeltildi (chown + chmod)
- Publish scripti vault-server alias'ını varsayılan kullanır
