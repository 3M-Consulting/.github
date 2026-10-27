# Sicurezza

Questo file vale per ogni repository di 3M Consulting che non ha un proprio `SECURITY.md`.

## Segnalare una vulnerabilità

**Non aprire una issue.** Scrivi a **tecnico@studio3mstp.com**.

Indica come riprodurre il problema, quale repository e quale superficie tocca, e che impatto ha sui dati. Non allegare dati personali di clienti o dipendenti: bastano gli identificativi tecnici.

Rispondiamo il prima possibile. I tempi della correzione e della divulgazione si concordano con chi ha segnalato.

## Fuori ambito

- **Dipendenze di terze parti**: la vulnerabilità si segnala al progetto a monte; poi avvisaci, se tocca anche noi.
- **Servizi di terzi** (GitHub, fornitori cloud): si segnalano al fornitore.

## Rilievi interni

Un rilievo di sicurezza già noto — difesa in profondità, configurazione, conformità — si registra come issue con il modulo «Registra un rilievo di sicurezza», che porta l'etichetta `security-review`. È quello, non un file, il registro dei rilievi aperti.

## Segreti

Una chiave finita nella storia di git va considerata compromessa: prima si ruota presso chi l'ha emessa, poi, se serve, si riscrive la storia. I controlli di igiene (gitleaks) la cercano in ogni pull request.
