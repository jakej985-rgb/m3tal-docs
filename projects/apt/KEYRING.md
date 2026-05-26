# Keyring & GPG Troubleshooting

If you encounter signing issues or signature mismatch errors while updating packages, follow these instructions to verify or reinstall the GPG key.

---

## Common Error: "The following signatures couldn't be verified"
This error typically means your package manager does not have the updated GPG signing key or has an incorrect path mapped in `/etc/apt/sources.list.d/m3tal.list`.

### Solution: Re-import Key
Remove any existing keyring and download a fresh copy:
```bash
sudo rm -f /usr/share/keyrings/m3tal-archive-keyring.gpg
curl -fsSL https://jakej985-rgb.github.io/m3tal-apt-key/public.key | sudo gpg --dearmor -o /usr/share/keyrings/m3tal-archive-keyring.gpg
sudo chmod 644 /usr/share/keyrings/m3tal-archive-keyring.gpg
```

---

## Verifying Key Details
You can inspect the imported GPG key signature by converting it to read-only format:
```bash
gpg --show-keys /usr/share/keyrings/m3tal-archive-keyring.gpg
```
Make sure the key issuer matches the M3TAL developer team email and that the expiration date is valid.

---

## Adjusting Repository List
Verify that `/etc/apt/sources.list.d/m3tal.list` contains the correct `signed-by` attribute pointing to the key:

```text
deb [signed-by=/usr/share/keyrings/m3tal-archive-keyring.gpg] https://jakej985-rgb.github.io/m3tal-apt-key stable main
```
If the path differs, update the file using a text editor.
```bash
sudo nano /etc/apt/sources.list.d/m3tal.list
```
After making modifications, run `sudo apt update` to reload package signatures.
