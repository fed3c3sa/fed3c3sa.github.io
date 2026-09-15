# Passaggio in produzione — checklist

Completa lo step 6 della guida *«Più account Gmail su Claude Desktop»*: elimina la scadenza delle
autorizzazioni a 7 giorni, senza sottoporre l'app a verifica.

Tempo: ~10 minuti in console, più ~1 minuto per casella di riautenticazione.

---

## Cosa cambia davvero

| | Modalità test (oggi) | In produzione, non verificata (dopo) |
| --- | --- | --- |
| Scadenza autorizzazioni | **7 giorni**, poi `invalid_grant` | **Nessuna** |
| Chi può autenticarsi | Solo gli indirizzi nella lista Utenti di test | Qualsiasi account Google, fino a 100 in totale |
| Schermata «Google non ha verificato questa app» | Compare | **Compare ancora** — è normale, si passa da *Avanzate* |
| Lista Utenti di test | Va mantenuta aggiornata | Diventa irrilevante |
| Costo | €0 | €0 |

Le due cose che **non** cambiano vanno messe in chiaro subito, per non aspettarsele: il warning di app
non verificata resta, e gli scope concessi restano ampi. Pubblicare non è verificare.

Fonti: [pubblicazione e cap utenti](https://support.google.com/cloud/answer/15549945) ·
[verifica non obbligatoria per uso personale](https://support.google.com/cloud/answer/13464323).

---

## Valori da incollare

Da usare identici in Google Cloud Console — il nome app deve corrispondere esattamente a quello scritto
nelle pagine del sito.

```
Nome applicazione     Personal Workspace Connector
Email di assistenza   (il tuo indirizzo Google, quello con cui hai creato il progetto)
Home page             https://fed3c3sa.github.io/
Privacy policy        https://fed3c3sa.github.io/privacy.html
Termini di servizio   https://fed3c3sa.github.io/terms.html
Dominio autorizzato   fed3c3sa.github.io
```

> Il dominio autorizzato è `fed3c3sa.github.io` per intero, **non** `github.io`: quest'ultimo è un
> public suffix e Google lo rifiuta. In genere la console lo compila da sola quando salvi gli URL.

---

## 1. Verifica che il sito risponda

Prima di toccare la console. Entrambe le pagine devono restituire `200` ed essere leggibili senza login,
da una finestra in incognito:

```sh
curl -sSI https://fed3c3sa.github.io/            | head -1
curl -sSI https://fed3c3sa.github.io/privacy.html | head -1
curl -sSI https://fed3c3sa.github.io/terms.html   | head -1
```

Se GitHub Pages è stato appena attivato, il primo deploy può richiedere qualche minuto e nel frattempo
risponde `404`. Aspetta e riprova prima di proseguire: un URL che non risponde è il modo più comune di
far fallire questo passaggio.

## 2. Branding

Console → [Google Auth Platform](https://console.cloud.google.com/auth/branding) → **Branding**.
Controlla in alto che sia selezionato il progetto giusto.

Compila i campi con i valori qui sopra e salva. Il logo è facoltativo: caricarne uno **attiva** la
necessità di verifica per poterlo mostrare, quindi per ora lasciatelo stare.

## 3. Pubblica

Console → **Pubblico** → riquadro *Stato di pubblicazione* → **Pubblica app** → conferma.

Lo stato passa da *Test* a *In produzione*. È immediato, non c'è nulla da attendere e nessuno da
avvisare.

## 4. Non premere «Prepara per la verifica»

È il bottone accanto, e sono due cose distinte. Sottoporre l'app alla verifica con scope Gmail
*restricted* apre un processo lungo che include una valutazione di sicurezza CASA — completamente
inutile per un'app usata da una persona sola. Se la console lo suggerisce, ignora il suggerimento.

## 5. Riautentica ogni casella (una volta sola)

Passaggio che si dimentica facilmente, e senza il quale sembra che il cambio non abbia funzionato: i
refresh token già in tuo possesso sono stati emessi **in modalità test**, quindi conservano la scadenza
a 7 giorni. Vanno sostituiti con token nuovi, emessi ora che l'app è in produzione.

Per ogni casella:

```sh
rm -rf ~/.workspace-mcp/personale/*
rm -rf ~/.workspace-mcp/lavoro/*
```

Poi riavvia Claude Desktop con `Cmd+Q` (non la X), chiedi in chat di leggere la posta di una casella,
apri il link di autorizzazione, approva. Ripeti per l'altra.

Usa una finestra in incognito diversa per ciascuna: alla seconda autenticazione il browser propone di
default l'account appena autorizzato, ed è l'errore più probabile dell'intera procedura.

## 6. Verifica finale

- [ ] `myaccount.google.com/connections` elenca **Personal Workspace Connector** per entrambe le caselle
- [ ] Claude legge messaggi diversi dalle due caselle
- [ ] In console, *Pubblico* riporta stato **In produzione**
- [ ] Segna sul calendario un controllo fra 10 giorni: se a quel punto non hai dovuto riautenticare
      nulla, la scadenza a 7 giorni è effettivamente sparita

Dopo la conferma puoi smettere di mantenere la lista Utenti di test, e cancellare l'eventuale
promemoria settimanale di riautenticazione.

---

## Se qualcosa va storto

**«Devi verificare la proprietà del dominio»** al salvataggio del Branding. La console a volte la
richiede. Si risolve in 5 minuti su [Search Console](https://search.google.com/search-console):
aggiungi una proprietà di tipo *Prefisso URL* con `https://fed3c3sa.github.io/`, scegli il metodo
**file HTML**, scarica il file `google….html`, copialo nella root di questo repo, `git push`, attendi il
deploy e premi *Verifica*. Usa lo stesso account Google del progetto Cloud.

**L'URL viene rifiutato come non valido.** Quasi sempre è il dominio autorizzato mancante o scritto
come `github.io` invece di `fed3c3sa.github.io`. Correggi quello e risalva.

**Continui a ricevere `invalid_grant` dopo qualche giorno.** Lo step 5 non è stato completato: stai
ancora usando un token emesso in modalità test. Svuota di nuovo la cartella credenziali e riautentica.

**Ti serve tornare indietro.** *Pubblico* → *Torna alla fase di test*. Reversibile in qualsiasi
momento, senza conseguenze; i token emessi in produzione smettono però di essere rinnovati senza
scadenza.

---

## Manutenzione

Se in futuro abiliti altri strumenti (Drive, Documenti, Tasks…) e questo amplia gli scope richiesti,
aggiorna `privacy.html` e `it/privacy.html` **prima** di usarli, e sposta la data di ultimo
aggiornamento. La tabella degli scope è già scritta larga apposta — copre Gmail, Calendar, Drive,
Documenti, Fogli, Presentazioni, Moduli, Tasks, Chat e Contatti — quindi nella maggior parte dei casi
non servirà toccare nulla.
