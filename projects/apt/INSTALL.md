# APT Repository Manual Setup

For systems where the automatic installation script is not used, you can add the M3TAL package repository manually.

---

## 1. Import Repository GPG Key
Download and save the public GPG key into your system's keyrings folder. This key signs the packages and verifies their integrity:

```bash
curl -fsSL https://jakej985-rgb.github.io/m3tal-apt-key/public.key | sudo gpg --dearmor -o /usr/share/keyrings/m3tal-archive-keyring.gpg
```

Verify that the key file is successfully generated:
```bash
file /usr/share/keyrings/m3tal-archive-keyring.gpg
```

---

## 2. Add Sources Entry
Create a sources entry pointing to the M3TAL repository. Make sure to specify the trusted keyring:

```bash
echo "deb [signed-by=/usr/share/keyrings/m3tal-archive-keyring.gpg] https://jakej985-rgb.github.io/m3tal-apt-key stable main" | sudo tee /etc/apt/sources.list.d/m3tal.list
```

---

## 3. Package Sync and Install
Update your package listings and install the core tools:

```bash
sudo apt update
sudo apt install -y m3tal
```
Once completed, you can run `m3tal` to launch the client.
