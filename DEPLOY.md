# Zepplin Studios — GitHub → one.com deploy

Elke `git push` naar `main` zet de site automatisch live op one.com. Eenmalige setup: ±15 minuten.

## Structuur

```
zepplin-studios-site/
├── site/                 ← alles hierin wordt gedeployed (webroot-spiegel)
│   └── index.html
├── .github/workflows/
│   └── deploy.yml        ← de deploy-pipeline
└── DEPLOY.md
```

Alleen de inhoud van `site/` gaat naar de server. Documentatie, notities, drafts: gewoon in de repo-root, die blijven op GitHub.

## Stap 1 — SFTP activeren op one.com

1. Log in op one.com → **Control Panel** → **SSH & SFTP**
2. Activeer SFTP en **zet een wachtwoord** (los van je one.com-login)
3. Noteer:
   - **Host**: staat erbij, formaat `ssh.zepplin.eu` (jouw domein)
   - **Gebruikersnaam**: staat erbij (vaak je domeinnaam of e-mailadres)

## Stap 2 — Webroot-pad bepalen

Open de **File Manager** op one.com en kijk waar de huidige `index.html` / placeholder staat:

- Staat die **in de root** van wat je ziet → remote path = `.`
- Staat die in een map (bv. `www` of je domeinnaam) → remote path = die mapnaam, bv. `/www`

Dit wordt straks het secret `SFTP_REMOTE_PATH`.

## Stap 3 — Repo aanmaken en pushen

```bash
cd zepplin-studios-site
git init -b main
git add .
git commit -m "Initial site + deploy pipeline"
gh repo create GLWD-24/zepplin-studios-site --private --source=. --push
# of zonder gh CLI: repo aanmaken op github.com en:
# git remote add origin git@github.com:GLWD-24/zepplin-studios-site.git
# git push -u origin main
```

## Stap 4 — Secrets instellen op GitHub

Repo → **Settings → Secrets and variables → Actions → New repository secret**, vier stuks:

| Secret              | Waarde                                  |
|---------------------|------------------------------------------|
| `SFTP_HOST`         | `ssh.zepplin.eu` (uit stap 1)     |
| `SFTP_USER`         | gebruikersnaam uit stap 1               |
| `SFTP_PASSWORD`     | het SFTP-wachtwoord uit stap 1          |
| `SFTP_REMOTE_PATH`  | `.` of `/www` (uit stap 2)              |

## Stap 5 — Eerste deploy

De push uit stap 3 heeft de Action al getriggerd (of trigger handmatig: **Actions-tab → Deploy naar one.com → Run workflow**). Groen vinkje = site live. Check je domein.

## Daarna: de dagelijkse flow

```bash
# aanpassing maken in site/index.html
git add . && git commit -m "Copy aanpassing hero" && git push
# ±40 seconden later staat het live
```

## Troubleshooting

- **Action faalt op login** → wachtwoord-secret opnieuw zetten (let op kopieerspaties); check dat SSH/SFTP actief is in het one.com-panel.
- **Deploy slaagt maar site verandert niet** → `SFTP_REMOTE_PATH` wijst naar de verkeerde map; vergelijk met de File Manager (stap 2). Ook: browsercache — test met Ctrl+Shift+R.
- **`mirror --delete` waarschuwing**: de workflow spiegelt `site/` exact naar de webroot en verwijdert wat er niet in staat. Staan er op one.com bestanden die moeten blijven (bv. een oud `send.php`), zet ze mee in `site/` of haal `--delete` uit de workflow.

## Nog te doen (los van deploy)

- [ ] Formulier koppelen (Formspree of `send.php` op one.com)
- [ ] `hallo@zepplin.eu` aanmaken onder Mail
- [ ] SSL "Force HTTPS" aanzetten in het one.com-panel
- [ ] Privacy- en juridische pagina's
