# Guía rápida: subir este repo a GitHub

## 1. Preparación (una sola vez)
```bash
sudo apt update && sudo apt install git -y
git config --global user.name "TuNombre"
git config --global user.email "tu@email.com"
git config --global init.defaultBranch main
ssh-keygen -t ed25519 -C "tu@email.com"
cat ~/.ssh/id_ed25519.pub
```
Copiá la clave pública en GitHub: Settings → SSH and GPG keys → New SSH key.
Probá con `ssh -T git@github.com`.

## 2. Crear el repositorio en GitHub
New repository → nombre `ccna-security-lab` → sin README (ya lo tenés).

## 3. Subir
```bash
cd ccna-security-lab
git init
git add .
git commit -m "Primer commit: estructura, notas y lab 01"
git remote add origin git@github.com:TU_USUARIO/ccna-security-lab.git
git push -u origin main
```

## 4. Flujo de trabajo para cada lab nuevo
```bash
git checkout -b lab-03-vlans
git add .
git commit -m "Lab 03: configuración de VLANs y trunk"
git push -u origin lab-03-vlans
```
Abrí un Pull Request en GitHub y hacé merge a `main`.

## 5. Antes de cada push
- [ ] Sin contraseñas reales ni `enable secret` verdaderos
- [ ] Sin IPs ni datos reales de tu red
- [ ] Capturas `.pcap` sanitizadas
- [ ] Sin documentos personales (DNI, certificados, etc.)

Revisá siempre con `git status` y `git diff --staged` qué vas a subir.
