[简体中文](README.md) | [English](README.en.md) | [Русский](README.ru.md) | [日本語](README.ja.md) | [한국어](README.ko.md) | [Español](README.es.md) | [العربية](README.ar.md) | [Čeština](README.cs.md) | [Dansk](README.da.md) | [Deutsch](README.de.md) | [Esperanto](README.eo.md) | [فارسی](README.fa.md) | [Suomi](README.fi.md) | [Français](README.fr.md) | [Ελληνικά](README.gr.md) | [Magyar](README.hu.md) | [Bahasa Indonesia](README.id.md) | [Italiano](README.it.md) | [മലയാളം](README.ml.md) | [Nederlands](README.nl.md) | [Norsk](README.no.md) | [Polski](README.pl.md) | [Português](README.ptbr.md) | [Română](README.ro.md) | [Türkçe](README.tr.md) | [Українська](README.ua.md) | [Tiếng Việt](README.vn.md)

# cloud-3xui

Δημιουργήστε έναν κόμβο 3x-ui (VLESS + WebSocket + TLS) σε δωρεάν διακομιστές GitHub Actions, με σταθερό domain και αυτόματη επανεκκίνηση.

## Πώς λειτουργεί

Κάθε εκτέλεση εγκαθιστά αυτόματα το πάνελ 3x-ui στον διακομιστή Ubuntu του GitHub, δημιουργεί ένα VLESS+WS inbound μέσω του API του πάνελ και εκθέτει το πάνελ και τον κόμβο μέσω **ονομασμένου tunnel** της Cloudflare (σταθερό domain). Κάθε εκτέλεση διαρκεί περίπου 5 ώρες και ενεργοποιεί αυτόματα την επόμενη πριν ολοκληρωθεί, οπότε η διεύθυνση του κόμβου δεν αλλάζει ποτέ.

## Σταθερές διευθύνσεις

- Κόμβος: `vless.my-github.ccwu.cc:443` (VLESS + WS + TLS)
- Πάνελ: `https://panel.my-github.ccwu.cc/panel` (όνομα χρήστη `admin`)

## Αρχική ρύθμιση

### 1. Ονομασμένο tunnel Cloudflare

1. Cloudflare Dashboard → Zero Trust → Networks → Tunnels → Create tunnel (Cloudflared)
2. Ονομάστε το tunnel και αντιγράψτε το token (τη συμβολοσειρά που ξεκινά με `eyJ`)
3. Προσθέστε δύο public hostnames στο tunnel:
   - `vless.to-domain-sas.com` → HTTP → `127.0.0.1:8080`
   - `panel.to-domain-sas.com` → HTTP → `127.0.0.1:2053`

### 2. GitHub Secrets

Στο αποθετήριο: Settings → Secrets and variables → Actions, προσθέστε τρία:

| Name | Value |
|------|-------|
| `CF_TUNNEL_TOKEN` | Token του tunnel Cloudflare |
| `PANEL_PASSWORD` | Κωδικός σύνδεσης στο πάνελ |
| `VLESS_UUID` | UUID του κόμβου (σταθερό, μόνιμα έγκυρο στον client, δημιουργήστε ένα μόνοι σας στο https://1024tools.com/uuid/, **Φυλάξτε αυτό το UUID, δεν εμφανίζεται στα logs, θα το χρειάζεστε κάθε φορά που συνθέτετε το link του κόμβου**) |

### 3. Εκτέλεση

Actions → 3x-ui VLESS Node → Run workflow (οι παράμετροι μπορούν να μείνουν κενές, τα secrets χρησιμοποιούνται αυτόματα).

Το αρχείο καταγραφής εκτυπώνει τον πλήρη σύνδεσμο `vless://`, ο οποίος μπορεί να εισαχθεί σε v2rayN / v2rayNG / Streisand / NekoBox.

## Αυτόματη επανεκκίνηση

- Κάθε εκτέλεση διαρκεί `run_minutes` λεπτά (προεπιλογή 290) και ενεργοποιεί αυτόματα την επόμενη κατά την ολοκλήρωση
- Το domain και το UUID είναι σταθερά, η ρύθμιση του client παραμένει μόνιμα έγκυρη
- Για διακοπή: πατήστε Cancel στα Actions, οι ακυρωμένες εκτελέσεις δεν επανεκκινούν αυτόματα

## Σημειώσεις

- **Ταχύτητα**: Η κίνηση περνά μέσω Cloudflare → κέντρα δεδομένων GitHub (ΗΠΑ/ΕΕ), κατάλληλο μόνο για προσωρινή/ελαφριά χρήση
- **Σύντομη διακοπή**: Περίπου 3-5 λεπτά διακοπής σε κάθε αυτόματη επανεκκίνηση (εκκίνηση νέου μηχανήματος, εγκατάσταση 3x-ui, επανασύνδεση tunnel), ο client επανασυνδέεται αυτόματα
- **Συμμόρφωση**: Η εκτέλεση proxy στο GitHub Actions μπορεί να παραβιάζει τους ToS με κίνδυνο αναστολής λογαριασμού, αξιολογήστε μόνοι σας
- **Μην** κοινοποιείτε δημόσια τα `CF_TUNNEL_TOKEN` / `PANEL_PASSWORD` / `VLESS_UUID`
